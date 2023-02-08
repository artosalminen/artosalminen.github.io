---
title: "Template, post header here"
date: 2023-02-09T10:00:00Z
series: ["Synology"]
weight: 1
tags: ["Synology"]
author: "Arto Salminen"
---

To access the Syonology NAS ports outside of your local network, you need to
set up DDNS, a wildcard certificate, and a reverse proxy to support HTTPS access.

## DDNS

Go to `Control Panel` / `External Access` / `DDNS`. Click **Add**.

Make the following selections:

- Service Provider: Synology
- Hostname: `yourname.synology.me`
- Username/Email: `<your email>`
- Password: `<make it up>`
- Exteral address: no need to change

## Wildcard certificate

Go to `Control Panel` / `Security` / `Certificate`. Click **Add**.

- Select replace existing certificate
- Select your Synology DDNS from the list
- Select Get a certificate from Let's Encrypt
- Check "Set as default certificate" and click Next

Configure the Let's Encrypt certificate lke this
- Domain name: `yourname.synology.me`
- Email: `<your email>`
- Subject Alternative Name: `*.yourname.synology.me` and click Done

## Reverse Proxy

Open the Synology Control Panel and navigate to `Login Portal` / `Advanced` / `Reverse Proxy`.
Then click `Create` to create a proxy.

Next, add a new rule for the proxy. Let's use the rule to access Codeserver port with
a subdomain hostname like `codeserver.yourname.synology.me`.

Set up the General tab configuration as follows:
- Reverse proxy name: `codeserver.yourname.synology.me`.
- Source Protocol: HTTPS
- Source hostname: `codeserver.yourname.synology.me`
- Port: 443
- Enable HSTS: checked
- Access control profile: Not set
- Destination protocol: HTTP
- Hostname: localhost
- Port: 8377

Set up Custom Header tab headers:

| Header name | Value               |
|-------------|---------------------|
| Upgrade     | $http_upgrade       |
| Connection  | $connection_upgrade |

