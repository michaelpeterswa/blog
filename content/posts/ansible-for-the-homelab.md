---
title: "Ansible for the Homelab"
date: 2022-02-13T17:29:55-08:00
tags: ["blog"]
author: "michaelpeterswa"

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true

draft: false
---
# Ansible for the Homelab

![Ansible Logo](https://i.pinimg.com/originals/10/66/ca/1066cabb206c21ff44dbb21434ed8e48.png)

It's been a while since I last posted. Well, I've been working on something special. With my first server (an HP laptop with a broken screen) being nearly five years old, I was rapidly growing concerned about the health of it's battery. The laptop has been plugged in and "floating" at 100% for the last 3-4 years. Given it's previous life as a daily driven laptop (~7 years of total use), I assumed one day in the near future it would decide to become a [spicy pillow](https://www.reddit.com/r/spicypillows/). I wanted to avoid a house fire, so I picked up a Dell Optiplex 7060 Micro on eBay as it's trusted replacement. It seemed like the perfect time to try something new with how I provision my servers, especially one that mostly remains in a static state. 

## Ansible Overview
Here's a great overview of the Ansible tool from the creators:

> Ansible is a radically simple IT automation platform that makes your applications and systems easier to deploy and maintain. Automate everything from code deployment to network configuration to cloud management, in a language that approaches plain English, using SSH, with no agents to install on remote systems.
> 
> — [Ansible GitHub Repository](https://github.com/ansible/ansible)

Ansible is often used for provisioning server hardware alongside tools such as Vagrant and Terraform. In my homelab, I wanted to use this technology to my advantage. Automation will be key to preventing manual configuration down the road.

## Operating Sytem Choice
Up to this point, I have manually configured all of my servers. I've tried a bit of everything, Raspbian, Ubuntu, Unraid, Arch, and even Amazon Linux. The newcomer to my current ecosystem is Rocky Linux. I was an early financial supporter of the Rocky Linux project because I wanted to support a no-frills enterprise focused Linux distribution. At this point, there's a noticible lack of documentation when compared to other distributions like Ubuntu. Luckily, because it's a derivative of RHEL, many of the commands for RHEL and CentOS are a drop-in replacement.

## Implementation
Over the course of the last month, I've [developed a handful of Ansible Playbooks](https://github.com/michaelpeterswa/playbooks/tree/main/rocky-edge) to automate every step (except installation of an OS) of my new edge server. This server only handles NGINX and some local testing where a linux terminal is needed. I wanted to ensure that it's easily repeatable in the event that I needed to start completely from scratch. It's a two-part process. The first playbook is designed to setup the SSH configuration and ensure that SELinux is configured correctly. The second (which now utilizes key-pair authentication), is designed to follow my normal installation process step-by-step. The process goes as follows:

1. Set up NTP Server connection to provide accurate time to the server (using Chrony)
2. Download and install all of the relevant packages (for both CLI and system use)
3. Download and configure Oh-My-ZSH with Antibody so that my terminal matches the rest of my computers
4. Install Docker
5. Install and configure Fail2Ban
6. Setup the AWS CLI (which is used for Route53)
7. Configure the Message Of The Day
8. Install and configure Certbot to use Route53 for primary DNS
9. Configure `firewalld` and SELinux to allow traffic on port 80 and 443 (http/s)
10. Spawn the NGINX container with my reverse proxy configurations

## Gotchas
Like many pieces of software, there is a right and wrong way of doing things. On first experience, I found it difficult to decouple the idea of running everything as a `ansible.builtin.shell` command. In Ansible, there are a plethora of modules that can be used to provide a more fluid experience. In particular, I utilized `dnf`, `community.general.lastpass`, `template`, and `service` the most. They fit my use cases quite well, and made the resulting YAML much smaller. These modules are great for abstracting away repeatable chunks of code.

## Endnote
I plan on writing more on this subject in the future, or at least taking a deeper dive into how Ansible can be leveraged for the home user. In the meantime, check out [Jeff Geerling](https://www.jeffgeerling.com/blog). His blog and YouTube Channel are an indespensable wealth of information about Raspberry Pi's, networking, homelab's, and general tech content.