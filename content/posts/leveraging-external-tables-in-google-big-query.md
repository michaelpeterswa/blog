---
title: "Leveraging External Tables in Google BigQuery"
date: 2022-06-29T13:09:15-07:00
tags: ["blog", "go", "google", "gcs", "bigquery"]
author: "michaelpeterswa"

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true

draft: false
---

## Context
Recently, I was tasked with logging error messages, request data, and the response body of failed HTTP requests to BigQuery for a service that makes outgoing `GET` and `POST` requests to thousands of different endpoints. Simple enough, right? 

## Loading Logs into BigQuery
Thankfully, the project already contained a legacy BigQuery streaming implementation which I used as my batch writer. After a thorough review of the changes, it was deployed to production. It worked well, almost too well. In the 10 minutes this change was live, it had written 4.5 million rows to BigQuery. I soon learned that we currently process nearly 10 times more traffic than is reported by our metrics dashboard. As it turned out, our metrics were counting incorrectly for years.  Publishing that many logs to BigQuery did not make financial sense, even if the rows were partitioned by hour and only kept for a short period of time (6-12 hours). The error logs produced by this service are mainly used as a diagnostic tool in situations where our service is experiencing degraded performance.

After determining that there wasn't a significant amount of duplicates within the data, I was confident that the traffic was legitimate and our service processed too much volume to be stored in BigQuery.

> NOTE: Systems utilizing message queues often cannot perform "exactly-once" delivery, and therefore have a small but expected amount of duplicate messages.

Luckily, we were also able to run some queries against the 10 minute slice of data to determine outlier error messages that could safely be dropped. Filtering out those error messages prior to insertion lead to about a 50% space savings, a great improvement.

## Back to the Drawing Board
BigQuery was originally selected as our storage for these logs because of the powerful ability to rapidly query the data to make inferences and determine trends. It didn't make good sense to stray away from the platform if our teams were already very familiar with it. We were beginning to run out of ideas for viable solutions, when I was reminded of the "[External Tables](https://cloud.google.com/bigquery/docs/external-tables)" feature of BigQuery. I had previously utilized external tables on a small scale a few months back for another solution we were building. Using Google Cloud Storage (GCS) as our object store, we could batch and write these files on an interval and use wildcards in our query to build the final external table.

Here's an example of that query:

```SQL
CREATE EXTERNAL TABLE `project_id.dataset_name.table_name`
OPTIONS (
	format = 'CSV',
	uris = ['gs://error-logs/2022/06/29/2022-06-29-h19*']
);
```

## Rewriting Logs Exporter for GCS
Luckily, much of my original implementation was reusable. I quickly wrote a test application to simulate writing a gzip-compressed CSV file to GCS. With simple random test data there were no problems writing a file to GCS and loading it into BigQuery. Things were looking good. I added each piece of the test file into the main program, testing as I went. The end was in sight as piece-by-piece the solution came together. It was important to configure all of the intervals and constants as environment variables/flags so that they could easily be modified as needed. 

With local testing showing promising results, it was time to deploy the changes.

## Initial Deployment
As soon as the new deployment was live, it was clear that something was very wrong. The pod logs were indicating that the service account in production did not have the `storage.objects.create` permission. Yikes! In error, I had aquired the permissions for a different service account than the one that was being used in production. I think it's time to update the documentation 🙂. Conveniently, when updating the IAM binding for testing, I used the role `roles/storage.objectCreator`. For this service though, we needed many other permissions, so I copied the set of permissions that weren't already in the custom binding. 

```
resourcemanager.projects.list
storage.objects.create
storage.multipartUploads.create
storage.multipartUploads.abort
storage.multipartUploads.listParts
```

As it turns out, the permission `resourcemanager.projects.list` is no longer valid (or at least no longer accepted by the Google API that Terraform interfaces with) which caused the service account to lose all permissions. How convenient! After determining why our logging queues were growing, I removed the erroneous permission and the queue quickly drained.

Data was flowing into GCS and seemed to be writing at the expected rate.

## External Table Woes
The next day, I tried for the first time to load a CSV into BigQuery that contained production traffic. Quickly, I learned that the schema was unable to be auto-detected. At first glance, it was obvious why. Because we log response body in full, some responses contain whitespace commonly found in HTML pages such as return and tab characters. After fixing that issue, the schema reappeared as expected. The table still failed to be usable though. Unbeknownst to me at the time, Google BigQuery External Tables require only valid and printable UTF-8 code points for the STRING type. The BigQuery error was indicating that there were invalid characters present in the CSV files. Although difficult to find, the `strings` package contains a convienient helper function, [strings.ToValidUTF8()](https://pkg.go.dev/strings#ToValidUTF8)  that ensures a given string only consists of valid UTF-8 characters. As far as I knew, the problems were now solved. That was not the case. I still was receiving errors that read:

> Error: Error detected while parsing row starting at position: {character number}. Error: Bad character (ASCII 0) encountered. 

I dug deeper into one of the raw CSV files and saw that there were some funny looking characters. Here's an example of that: [GIF89a����������!���,�������L�;](https://apps.timwhitlock.info/unicode/inspect?s=GIF89a%01%00%01%00%EF%BF%BD%01%00%EF%BF%BD%EF%BF%BD%EF%BF%BD%00%00%00%21%EF%BF%BD%04%01%00%01%00%2C%00%00%00%00%01%00%01%00%00%02%02L%01%00%3B) I didn't realize it initially, but that sequence of characters is the headers of the `GIF89a` specification (of the popular `.gif` image format). A neat in-depth explanation can be found [here](https://www.matthewflickinger.com/lab/whatsinagif/bits_and_bytes.asp), and helped show me how the GIF format works under the hood. When the GIF header is decoded as UTF-8 it becomes littered with Unicode control characters, most notably, `NULL` and `START OF HEADING` (0x00 and 0x01) respectively. These invisible control characters were the source of BigQuery's vague error message. The `unicode` package in Go does have a function to detect non-printable runes  (which includes all control characters, except space). To remedy the issue, a function was implemented to only allow printable characters to exist in a given string. This was applied to all strings in the logs, and provided the final solution to this strange course of investigation.

```go
// cleanString ensures characters in a string are printable (e.g not "control" characters)
func cleanString(s string) string {
	return strings.Map(func(r rune) rune {
		if unicode.IsPrint(r) {
			return r
		}
		return -1
	}, s)
}
```

## Parting Thoughts
Often, solutions in software may look easy at first glance. Many times, it evolves into something much more complicated, whether limited by performance, cost, or architecture. It is important to stay flexible and always think of possible alternatives, you never know when the situation may change. Persistance is also a valuable trait. There were many times over the course of this task where I didn't see a successful outcome in sight, but I kept going. Eventually, an acceptable solution was found and I learned many valuable tips and tricks along the way.

