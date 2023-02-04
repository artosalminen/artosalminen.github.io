---
title: "How to Set Up a Codeserver on Synology NAS"
date: 2023-02-07T10:00:00Z
series: ["Synology", "Coding"]
weight: 1
tags: ["Synology", "Coding", "VSCode"]
author: "Arto Salminen"
draft: true
---

Have you ever missed the VSCode on your tablet? Now there is a solution: run the
VSCode on your NAS at home and access it with a browser. That will be possible when you run
[the Codeserver Docker container](https://registry.hub.docker.com/r/linuxserver/code-server/) on your NAS.

Before you do this, you probably want to set up a wildcard certificate and a proxy server on your NAS. That will
enable you to access the NAS outside your local network using HTTPS. How to do it, check this: 
[How to Set Up Wildcard Certificate and Reverse Proxy on Synology NAS]({{< ref "how-to-set-up-wildcard-certificate-and-proxy-on-synology-nas.md" >}})

You will also need to find out your UID and GID. Here are instructions to get those:
[How to find out your UID and GID on Synology NAS]({{< ref "how-to-find-uid-and-gid-on-synology-nas.md" >}})

And after you have sorted those things, you can set up the Codeserver as follows:

1. Install Docker to the Synology NAS from Package Center
2. Create a folder to map the codeserver files into, `docker/codeserver` works fine
3. Open the Docker app, search for [linuxserver/code-server](https://registry.hub.docker.com/r/linuxserver/code-server/) 
   from the registry, and download the image (latest)
4. Set the port mapping as 8377:8443
5. Adjust the advanced settings as follows:
   ```
   PUID: <your UID>
   PGID: <your GID>
   PROXY_DOMAIN: codeserver.<your DDNS domain>
   PASSWORD: <your very complex password>
   SUDO_PASSWORD: <your very complex password>
   TZ: <your timezone, e.g Europe/Helsinki>
   ```
6. Map the /config to the folder you created earlier (`docker/codeserver`)
7. Set automatic restart
8. Run the container

Now you should be able to navigate to `codeserver.<your DDNS domain>` enter the password
and enjoy your fresh VSCode on server.