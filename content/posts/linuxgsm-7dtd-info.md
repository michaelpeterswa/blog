---
title: "LinuxGSM 7DaysToDie Tips"
date: 2021-11-20T23:20:59-08:00
tags: ["blog", "gaming"]
author: "michaelpeterswa"

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true

draft: false
---
## Issue
This evening, I was struggling to access my new "7 Days To Die" server after provisioning a new Rocky Linux 8 VM to host it on. As a preface, I have plenty of experience hosting both bare-metal and VM game servers. For Steam games, my tool of choice is typically [LinuxGSM](https://github.com/GameServerManagers/LinuxGSM/) by Daniel Gibbs. This is a fabulous piece of software that abstracts some of the nitty-gritty away (which definitely helps when running servers from multiple games).
## Port-Forwarding
Normally, I would chalk a connection issue up to a misconfigured port forward.  I quickly determined that another issue was at fault when I couldn't access the server from the LAN. That is a telltale sign something else is at play.
## Google Searching
After many dead ends, I stumbled upon this [r/7daystodie thread](https://www.reddit.com/r/7daystodie/comments/n10crs/solution_for_timed_out_issue_on_a_dedicated/) where the OP noticed that there is a parameter in the settings file that prevents `SteamNetworking` from being enabled. I was dumbfounded that a default setting in the server config prevents networking/connection on the main platform of distribution.
## Important Settings
On line 15 of my configuration file (`sdtdserver.xml`), I removed `SteamNetworking` from the value field. This was based upon the suggestions from the above Reddit thread. 
```XML
<property name="ServerDisabledNetworkProtocols" value="SteamNetworking"/>      <!-- Networking protocols that should not be used. Separated by comma. Possible values: LiteNetLib, SteamNetworking. Dedicated servers should disable SteamNetworking if there is no NAT router in between your users and the server or when port-forwarding is set up correctly -->
```

## Wrap-up
I found that, when using a text-editor such as `nano` to edit the configuration file, the comment is obstructed which explains that SteamNetworking should be removed when "port-forwarding is set up correctly". I would definitely say that this *is* user error, but from a developer side, it could definitely be more clear that this value needs to be modified before use (without relying on line-wrap). This value is easily missed and there is no emphasis or clarity that a server won't be accessable unless you pay close attention to the configuration.