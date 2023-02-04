---
title: "How to find out your UID and GID on Synology NAS"
date: 2023-02-05T10:00:00Z
series: ["Synology"]
weight: 1
tags: ["Synology"]
author: "Arto Salminen"
draft: true
---

The UID and GID values for default user on Synlogy NAS are usually 1026 and 100.
There is a very simple way to check the values as follows:

1. Create a new Scheduled task asn User-defined script
2. Name the script whatever you see fit
3. Set it to be not repeating
4. Set it to send run details to your email
5. Write the script:
   ```
   id
   ```
   Not too complex, huh?
6. Run it and soon enough, you will receive the email containing the UID and GID values.

These values will be needed to install some Docker containers, for instance the Codeserver.