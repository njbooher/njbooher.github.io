---
#
# By default, content added below the "---" mark will appear in the home page
# between the top bar and the list of recent posts.
# To change the home page layout, edit the _layouts/home.html file.
# See: https://jekyllrb.com/docs/themes/#overriding-theme-defaults
#
layout: default
---

## Research and Writeups

### [Solr Query Injection](/security/solr-query-injection)

Apache Solr is a NoSQL datastore that is typically used as an enterprise search platform. Most resources on Solr injection seem to be about CVEs or injecting additional parameters into calls to the Solr REST API. I discovered several techniques to extract data with only the query term parameter.

### [Cloudflare-wide IP spoofing](/security/cloudflare-workers-ip-spoofing)

A vulnerability in Cloudflare Workers, a server-less edge computing service, allowed the worker to use your IP address when making requests to other sites behind Cloudflare, bypassing firewall rules.

### [Drupal core PHP code injection (DRUPAL-SA-CORE-2018-006)](/security/drupal-core-php-code-injection)

The Drupal core Contextual Links module didn’t sufficiently validate the requested contextual links. This allowed a render array to be injected, enabling an attacker to execute arbitrary PHP code.

## Other Advisories

### [Drupal core file metadata disclosure (DRUPAL-SA-CORE-2020-011)](https://www.drupal.org/sa-core-2020-011)

The Drupal core File module allowed an attacker to gain access to the file metadata of a permanent private file that they do not have access to.

### [Tripal BLAST UI shell code injection (DRUPAL-SA-CONTRIB-2016-054)](https://www.drupal.org/forum/newsletters/security-advisories-for-contributed-projects/2016-10-26/tripal-blast-ui-highly)

The Tripal Blast UI module didn’t sufficiently validate advanced options available to users submitting BLAST jobs, thereby exposing the ability to enter a short snippet of shell code that would execute when the BLAST job was run.