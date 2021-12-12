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
[GuildDarts on GitHub](https://github.com/GuildDarts/) for creating the incredible `Docker Folders` plugin for Unraid (which greatly helped me clean up my Docker container list). I found out about this plugin after creating a new IPFS container which rounded out a cluttered ~30 containers running on my Dell R620. With it, I was able to make a new `Web3` category to store my new related containers in. Now I have 6 or so categories to store my other containers in, and it is much more readable.

## IPFS Introduction
As described on their website, IPFS is:
> A peer-to-peer hypermedia protocol designed to preserve and grow humanity's knowledge by making the web upgradeable, resilient, and more open.

It's a neat piece of software written in Go that is powering some of the early work in what the community is calling *Web3*. Essentially, IPFS is a distributed storage network, which allows access to any node on the network. By definition, it's also decentralized. All the different pieces of data are spread out (distributed) across the network nodes.

## So How Does It Work?
I won't pretend to understand the finer and more technical details of this project (yet), but [their documentation](https://docs.ipfs.io/concepts/how-ipfs-works/#content-addressing) (which coincedentally also uses IPFS to access) does an admirable job of summing up the three key points that make IPFS tick. They are as follows:

1. Unique identification via content addressing
2. Content linking via directed acyclic graphs (DAGs)
3. Content discovery via distributed hash tables (DHTs)

You can also read more [here](https://docs.ipfs.io/concepts/).

## What Am I Doing Here?
I've recently become interested in exploring Blockchain, Crypto, and Web3 topics, and I wanted to explore the process of purchasing a domain from the Ethereum Name Service (which can be used to send Ethereum directly to me). In addition, using the `ContentHash` record, I wanted to host a simple HTML website from it. Hosting a file with an ENS (Ethereum Name Service) address can leverage IPFS to permanently (as long as it's shared) host a website. What's neat about this process is that it's completely resistant to censorship, whether it be from parents, coworkers, employers, or even government agencies. Yeah that's right, come at me NSA! I mean business. I'm surely already on one of your lists ([NSA labels Linux Journal readers and Tor and Tails users as extremists](https://www.digitaltrends.com/computing/nsa-labels-linux-tails-users-extremists/)).

## Hosting IPFS Container
The IPFS Desktop app conveniently provides the ability to access the IPFS network on your own computer. It does pose a problem though for persisting information on the network. IPFS coins the term *pinning* to describe the process of preventing the IPFS Node's garbage collector from removing a specific file or files. Using pinning on the Desktop version of IPFS is great, but it has one major downfall. If the file hasn't been shared yet, when your computer turns off, it will disappear from the IPFS network. This is where Docker comes in! I was able to provision an IPFS Docker Node following the instructions on their documentation, which can be found [here](https://docs.ipfs.io/how-to/run-ipfs-inside-docker/).

### IPFS on Docker Hints
The IPFS Docker container requires one port to be forwarded (4001 TCP/UDP) and also requires passing port 5001 to access the WebUI. Once started, you'll need to access the WebUI at the following address `http://<container-ip-address>:5001/webui` as the root (without `webui`) will return a 404. Then, it's incredibly important to follow the instructions provided on the WebUI to gain access to the IPFS API (by default it has very strict CORS requirements).

## Uploading HTML Website
The next step was to create and upload my simple HTML website to IPFS. In the Web3 mindset, I didn't want to rely on any Web2 (conventional) services for my site. With that, the font is included (courtesy of Google Fonts) and the HTML/CSS files have no external scripts or dependencies. After starting my Docker IPFS Node, I was ready to begin the upload process. The easiest way I have found is to use the IPFS Desktop App alongside the Docker container to provide permanent pinning (without the use of a pinning service like [Piñata](https://www.pinata.cloud/)). 

My website has multiple files (a font, a stylesheet, and the HTML document), so the easiest way to upload this to IPFS is with a folder. I went through this process using the Windows IPFS Desktop application. It's a pretty straightforward process, and gives you the option to select specific files within the folder to include/exclude in the final hash. Once the uploading process completes, you'll be provided with. 

For my Mac/Linux friends out there, the CLI (Command Line Interface) method to upload multiple files is also shown on the IPFS Documentation. It also allows for passing in an ignore file (so that you can ignore `.git/` and other files). I plan to explore the CLI interface in the future as I don't like to rely on Desktop applications when there's a suitable alternative available.

## Updating ContentHash
For an Ethereum Name Service domain, the method of attributing some piece of distributed content is by setting it in the ContentHash record. Currently, the whole process is quite expensive (purchasing, registering, and updating an ENS domain) as the Ethereum network is quite congested. You can check the current Ethereum Gas fee here on [ethgas.watch](https://ethgas.watch/). Throughout the process, I paid between $60 and $80 USD in transaction fees. Uh oh!

Once I was ready, I grabbed the CID of my folder (which is now stored on IPFS) and logged into the Ethereum Name Service's domain configuration page. From there, you are able to set the Content value for your domain, which in my case was an IPFS link. I then confirmed the transaction and paid the gas fee to have this ContentHash added to the Ethereum Blockchain. Unfortunately, at the time I didn't know that it's a good idea to use the IPNS (InterPlanetary Name Service) address, which allows future updates to the content. An example of that process can be found [here](https://docs.ipfs.io/concepts/ipns/#example-ipns-setup-with-js-sdk-api). For the time being, my website's content is unmodifiable and connected to my ENS Address.

## Accessing New IPFS Content
There are multiple ways to access the website now! Two links that should work in normal browsers are:

* https://alpin3.eth.link
* https://alpin3.eth.limo

The content is also accessible through IPFS at the link below:


## Final Notes
This process has been a ton of fun to follow and learn about (except for those darn ⛽ fees) and I look forward to spending more time in the blockchain space in the future.

Happy Hacking!
