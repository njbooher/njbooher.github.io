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

### [PHP code injection on Steamworks financial reporting site](https://hackerone.com/reports/518348)

Insufficient validation of parameters allowed an attacker to achieve arbitrary PHP code execution.

## Other Injections

### [SQL injection in Steamworks developer portal](https://hackerone.com/reports/690349)

A SQL injection vulnerability allowed queries against a legacy backing store.

### [Remote procedure call injection in Steamworks developer portal](https://hackerone.com/reports/652649)

Insufficient validation of parameters allowed making arbitrary calls to an internal data service.

## Access Control Issues

### [Microtransaction sales data](https://hackerone.com/reports/975212)

An authenticated partner could view microtransaction sales data for other publisher's games.

### [Steamworks Web APIs with the Dota2 API key](https://hackerone.com/reports/674800)

Path traversal enabled calling arbitrary Steamworks Web API methods using an API key that had elevated privileges for Dota2.

### [Other publisher's depots](https://hackerone.com/reports/1018368)

A parameter-validation error allowed publishers to add other publisher's depots to their own app.

### [Server config / API keys for Source game servers](https://hackerone.com/reports/1168557)

Unauthenticated visitors could see sensitive configuration information about Source game servers.

### [Steam Community Market transactions](https://hackerone.com/reports/577584)

Anyone with a partner key could make changes to Steam economy items, like trading cards, and reverse wallet fund spending on the Steam Community Market.

### [Generate CD Keys for the Steam Master package](https://hackerone.com/reports/972243)

Partners could add their own apps to certain Valve administrative packages, which could be further leveraged to generate CD key ranges for those administrative packages.