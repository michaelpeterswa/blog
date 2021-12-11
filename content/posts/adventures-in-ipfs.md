---
title: "Adventures in IPFS"
date: 2021-12-11T00:16:00-08:00
tags: ["blog", "web3", "crypto", "ipfs"]
author: "michaelpeterswa"

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true

draft: true
---

## Preface (just a shoutout)
Firstly, before digging into the real meat of this post, I'd like to thank 
[GuildDarts on GitHub](https://github.com/GuildDarts/) for creating the incredible `Docker Folders` plugin for Unraid (which greatly helped me clean up my Docker container list). I found out about this plugin after creating a new IPFS container which rounded out a cluttered ~30 containers running on my Dell R620. With it, I was able to make a new `Web3` category to store my new related containers in. I was able to make 6 or so categories to store my other containers in, and now it is much more readable.

## IPFS Introduction
As described on their website, IPFS is:
> A peer-to-peer hypermedia protocol designed to preserve and grow humanity's knowledge by making the web upgradeable, resilient, and more open.

It's a neat piece of software written in Go that is powering some of the early work in what the community is calling *Web3*. Essentially, IPFS is a distributed storage network, which allows access to any node on the network. By definition, it's also decentralized. All the different pieces of data are spread out (distributed) across the network nodes.

## So How Does It Work?
I won't pretend to understand the finer and more technical details of this project (yet), but [their documentation](http://docs.ipfs.io.ipns.localhost:8080/concepts/how-ipfs-works/#content-addressing) (which coincedentally also requires IPFS to access) does an admirable job of summing up the three key points that make IPFS tick. They are as follows:

1. Unique identification via content addressing
2. Content linking via directed acyclic graphs (DAGs)
3. Content discovery via distributed hash tables (DHTs)

You can also read more [here](http://docs.ipfs.io.ipns.localhost:8080/concepts/).

## What Am I Doing Here?
I've recently become interested in exploring Blockchain, Crypto, and Web3 topics, and I wanted to explore the process of purchasing a domain from the Ethereum Name Service (which can be used to send Ethereum directly to me). In addition, using the `ContentHash`, I wanted to host a simple HTML website from it. Hosting a file with an ENS (Ethereum Name Service) address can leverage the IPFS to permanently (as long as it's shared) host a website. What's neat about this process is that it's completely resistant to censorship, whether it be from parents, coworkers, employers, or even government agencies. Yeah that's right, come at me NSA! I mean business. I'm surely already on one of your lists ([NSA labels Linux Journal readers and Tor and Tails users as extremists](https://www.digitaltrends.com/computing/nsa-labels-linux-tails-users-extremists/)).

## Hosting IPFS Container

## Uploading HTML Site

## Updating ContentHash

## Accessing New IPFS Content

