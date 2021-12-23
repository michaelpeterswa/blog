---
title: "Hashes and Strings in Go"
date: 2021-12-22T18:57:31-08:00
tags: ["blog", "golang", "perf", "benchmark"]
author: "michaelpeterswa"

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true

draft: false
---

## Introduction

In a few cases while working on Go software, I have needed to generate a cryptographic hash (mainly SHA-256) from a string or JSON value. In all cases up to this point, I needed to convert the resulting `[]byte` to a string representation for storage or viewing. In my investigation, I found two popular ways to approach this problem. I'll describe them below as well as explore the performance implications of each choice.

## Generating the Hash
For all intents and purposes, this process will yield the same result for all common hashes such as MD5, SHA-1, SHA-256, and SHA-512. In my preliminary investigation, I only ran benchmarks on SHA-256. That benchmark data will be what we take a look at today. The process used is quite straightforward, and can be found [here](https://github.com/michaelpeterswa/sha_hex_string_bench/blob/main/shahexstringbench.go).

Of note, the line  `shaBytes := sha256.Sum256([]byte(a))` is supplied with a string of 20 random characters in each test.

## Method #1 (fmt.Sprintf)
```go
func MethodOne(a string) string {
	shaBytes := sha256.Sum256([]byte(a))
	return fmt.Sprintf("%x", shaBytes)
}
```
This was the first method that I encountered in production code at work. Using the `%x` (hexadecimal) format specifier in conjunction with `fmt.Sprintf()` is a quick way to return the string representation of a hash. In my opinion, this method is more useful when printing to the console with `fmt.Printf()` as it requires one less line of code than the second method.

## Method #2 (hex.EncodeToString)
```go
func MethodTwo(a string) string {
	shaBytes := sha256.Sum256([]byte(a))
	return hex.EncodeToString(shaBytes[:])
}
```
This was the method I settled upon to generate over 300,000 hashes-per-minute in multiple production services. When using this method it is important to remember that the `EncodeToString()` function requires a byte slice as it's argument. To convert the byte array to a byte slice is as simple as using `shaBytes[:]`, which can be seen above. 
## Performance Statistics
Below is the direct output of the benchmarking process.
```
λ  sha_hex_string_bench main ✗  go test -run=XXX -bench=. -benchtime=100000x
goos: darwin
goarch: amd64
pkg: github.com/michaelpeterswa/sha_hex_string_bench
cpu: Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
BenchmarkShaHexStringWithHexLib-12     	  100000	       729.5 ns/op	     176 B/op	       4 allocs/op
BenchmarkShaHexStringWithSprintf-12    	  100000	       966.3 ns/op	     176 B/op	       5 allocs/op
BenchmarkRandStringBytes-12            	  100000	       382.2 ns/op	      48 B/op	       2 allocs/op
PASS
ok  	github.com/michaelpeterswa/sha_hex_string_bench	0.336s
```
And here is the simplified (and arguably more readable) table.

| Benchmarks                      | iterations | ns/op | B/op | allocs/op |
|---------------------------------|------------|-------|------|-----------|
| Method One (fmt.Sprintf)        | 100,000    | 966.3 | 176  | 5         |
| Method Two (hex.EncodeToString) | 100,000    | 729.5 | 176  | 4         |
| Input String Generation         | 100,000    | 382.2 | 48   | 2         |

## Conclusion
As seen in the above data, `hex.EncodeToString()`is significantly faster. This is because the `EncodeToString` function does not rely on reflection internally like `Sprintf` does. It also has one less allocation to the heap for the same memory footprint, which is a benefit at scale. Although in most cases the added performance benefit wouldn't matter, it's important to take a step back and reflect on why certain design choices were made. Even small additions such as these SHA-256 hashes may negatively impact performance or increase cost in the long term. We want to avoid that! 😁

For more info about allocations and what they translate to under the hood check out this article: [Understanding Allocations in Go](https://medium.com/eureka-engineering/understanding-allocations-in-go-stack-heap-memory-9a2631b5035d)