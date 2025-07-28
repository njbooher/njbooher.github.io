---
layout: default
title: Nick Booher
---

## Security Research and Writeups

### [Solr Query Injection](/security/solr-query-injection)

Apache Solr is a NoSQL datastore that is typically used as an enterprise search platform. Most resources on Solr injection seem to be about CVEs or injecting additional parameters into calls to the Solr REST API. I discovered several techniques to extract data with only the query term parameter.

### [Cloudflare Workers IP Spoofing](/security/cloudflare-workers-ip-spoofing)

A vulnerability in Cloudflare Workers, a server-less edge computing service, allowed the worker to use your IP address when making requests to other sites behind Cloudflare, bypassing firewall rules.

### [Highlights From Hacking Steam](/security/steam-highlights)

I spent a lot of time from 2019-2021 hacking on Steam. This article has some of the highlights.

### [Drupal core PHP code injection](/security/drupal-core-php-code-injection)

The Drupal core Contextual Links module didn't sufficiently validate the requested contextual links. This allowed a render array to be injected, enabling an attacker to execute arbitrary PHP code.

## Other Security Advisories

### [Drupal core file metadata disclosure](https://www.drupal.org/sa-core-2020-011)

The Drupal core File module allowed an attacker to gain access to the file metadata of a permanent private file that they do not have access to.

### [Tripal BLAST UI shell code injection](https://www.drupal.org/forum/newsletters/security-advisories-for-contributed-projects/2016-10-26/tripal-blast-ui-highly)

The Tripal Blast UI module didn't sufficiently validate advanced options available to users submitting BLAST jobs, thereby exposing the ability to enter a short snippet of shell code that would execute when the BLAST job was run.