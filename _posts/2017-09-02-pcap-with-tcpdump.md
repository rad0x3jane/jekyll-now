---
layout: post
title: reading pcap files with tcpdump
---

***Step one***: Find a resource. My preferred method: have both the man page and a [tutorial](https://danielmiessler.com/study/tcpdump/) up, since they come at it from different angles.

Funny note from the man page:
(N.B.:The following description assumes familiarity with the TCP protocol  described  in RFC-793.  If you are not familiar with the protocol, neither this description nor tcpdump will be of much use to you.)

***Step two***: Open the file.
```tcpdump -r data.pcap```

In my case, I have to run it with sudo, otherwise I get "command not found". This happens with ifconfig too on Kali, haven't figured out why yet.

***Step three***: Read some packets.

