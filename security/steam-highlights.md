---
layout: default
title: Highlights From Hacking Steam
categories: security
---

I spent a lot of time from 2019-2021 hacking on Steam. This article has some of the highlights.

## RCE

### Shell injection on Steamworks developer portal

Insufficient validation of parameters allowed injecting shell metacharacters into values used by the backend in a Bash command.

* https://hackerone.com/reports/840243
* https://hackerone.com/reports/926169
* https://hackerone.com/reports/949361

### PHP code injection on Steamworks financial reporting site

Insufficient validation of parameters allowed an attacker to achieve arbitrary PHP code execution.

https://hackerone.com/reports/518348

## Other Injections

### SQL injection in Steamworks developer portal

A SQL injection vulnerability allowed queries against a legacy backing store.

https://hackerone.com/reports/690349

### Remote procedure call injection in Steamworks developer portal

Insufficient validation of parameters allowed making arbitrary calls to an internal data service.

https://hackerone.com/reports/652649

## Access Control Issues

### Microtransaction sales data

An authenticated partner could view microtransaction sales data for other publisher's games.

https://hackerone.com/reports/975212

### Steamworks Web APIs with the Dota2 API key

Path traversal enabled calling arbitrary Steamworks Web API methods using an API key that had elevated privileges for Dota2.
 
https://hackerone.com/reports/674800

### Other publisher's depots

A parameter-validation error allowed publishers to add other publisher's depots to their own app.

https://hackerone.com/reports/1018368

### Server config / API keys for Source game servers

Unauthenticated visitors could see sensitive configuration information about Source game servers.

https://hackerone.com/reports/1168557

### Steam Community Market transactions

Anyone with a partner key could make changes to Steam economy items, like trading cards, and reverse wallet fund spending on the Steam Community Market.

https://hackerone.com/reports/577584

### Generate CD Keys for the Steam Master package

Partners could add their own apps to certain Valve administrative packages, which could be further leveraged to generate CD key ranges for those administrative packages.

https://hackerone.com/reports/972243