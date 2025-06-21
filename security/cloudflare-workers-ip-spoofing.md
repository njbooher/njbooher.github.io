---
layout: default
title: Cloudflare Workers IP Spoofing
categories: security
---

## Cloudflare

Cloudflare is a distributed CDN and WAF. Customers configure their servers to only accept connections from Cloudflare servers and then change their DNS to point to Cloudflare's edge servers. Cloudflare proxies requests to the customer's backend servers, adding the [CF-Connecting-IP](https://support.cloudflare.com/hc/en-us/articles/200170986-How-does-Cloudflare-handle-HTTP-Request-headers-) header to identify the original visitor.

## Cloudflare workers

[Cloudflare Workers](https://workers.cloudflare.com/) is a serverless code hosting service. The service allows you to [deploy Javascript to Cloudflare's edge servers](https://developers.cloudflare.com/workers/learning/how-workers-works) that runs in the v8 runtime. A simple hello world worker looks like this:

```javascript
addEventListener("fetch", event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  return new Response("Hello world")
}
```

Worker code can make requests to external sites in the event handler.

```javascript
async function handleRequest(request) {
    return fetch(new Request('https://example.com'));
}
```

The outgoing request is made by either Google Compute Engine (dev) or Cloudflare's edge server (production), and the `CF-Connecting-IP` header on the outgoing request is set to an IPv6 address representing the worker. The Javascript code is not allowed to modify this header.

## The Bug

While experimenting with sample code from the documentation, I encountered a sample like this:

```javascript
async function handleRequest(request) {
    return fetch(new Request('https://example.com', request));
}
```

Note the addition of the second parameter to `Request`. The [Workers documentation](https://developers.cloudflare.com/workers/runtime-apis/request#parameters) indicates this is an optional object with settings to apply to the Request.

I noticed that when I pointed this sample at another Cloudflare worker, my IP address was sent in the `CF-Connecting-IP` header rather than the fixed IPv6 address normally set by the worker.

Further testing with the [/cdn-cgi/trace magic route](https://www.cloudflare.com/cdn-cgi/trace) indicated this wasn't specific to Cloudflare Workers. Sites behind Cloudflare's CDN and WAF also received my IP in the CF-Connecting-IP header for requests that my worker code made.

This behavior held even when my worker code modified query and POST parameters, as long as it didn't tamper with the request headers.

## Impact
Normally, when you visit a site protected by Cloudflare and send a request containing a `CF-Connecting-IP` header, Cloudflare drops the header and replaces it with your actual IP before sending it to the origin server.

However, when a Cloudflare worker makes a request to a site protected by Cloudflare, Cloudflare trusts the value of the header provided by the worker and passes it on to the origin server.

Thanks to this bug, if you made any request to my Cloudflare worker, even through an image embedded on a page, my worker could then make requests to any other site behind Cloudflare and pretend to have your IP address.

Some sites may make additional functionality available to users from specific IP addresses. Using the [Workers KV](https://developers.cloudflare.com/workers/learning/how-kv-works) data store, I could cache the response to the sub-request to check for that.

## Timeline
* June 17, 2020 09:45 AM - Reported to Cloudflare
* June 17, 2020 10:04 AM - Acknowledged by Cloudflare
* June 19, 2020 - Bounty received from Cloudflare
* June 24, 2020 - Fix committed by Cloudflare and deployed globally
* January 5, 2021 - Published with approval of Cloudflare
