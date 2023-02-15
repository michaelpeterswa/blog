---
title: "Observability in the Homelab"
date: 2022-07-17T18:06:20-07:00
tags: ["blog", "unraid", "docker", "prometheus", "loki", "grafana"]
author: "michaelpeterswa"

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true

draft: true
---
## What is a "Homelab"?
As usually defined on the internet, a "homelab" is a server, or collection of servers used for experimentation and home use. They are usually built up over time and can be as small as a Raspberry Pi or as large as a full-sized server rack with many machines and network devices.

## Homelab Monitoring

Many times, it is not possible, nor convenient to monitor all aspects of a homelab. It's never the first thing people think of doing either. When you start running more mission-critical services, it can be helpful to gain more insights into how well the machines, services, and containers are running. Over time, I picked up more methods of monitoring my homelab and I'll share these methods below.

## Early Stages
When I first began running servers at home, I relied on Ubuntu 16.04LTS Server. Some of the first pieces of software I ever installed were InfluxDB (a time-series database) and Grafana (a data analytics and visualization platform). These two programs are among the best free and open-source metrics gathering tools. When combined with Telegraf (another InfluxData product), server metrics such as CPU and memory usage can be easily gathered. This was a great first step into seeing how much proccessing power I was using at a given time. It is also quite easy to send data to InfluxDB from software using any of the [14 client libraries](https://docs.influxdata.com/influxdb/v2.3/api-guide/client-libraries/) currently supported.

## Complexity Increases
After many years of running Ubuntu on bare-metal, I decided it was time for a change. I loaded up Unraid (Lime Technologies) onto my Dell R620 and had a sigh of relief when I was able to interact through a simple GUI instead of relying upon just the command line. Quite frankly, I didn't have enough time to deal with my servers between school assignments.