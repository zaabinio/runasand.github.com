---
layout: post
title: "Testing Tor Hidden Services with Burp Pro"
date: 2013-02-25 09:33
comments: true
categories: security 
---

On February 15, Dafydd Stuttard announced the release of [Burp Suite v1.5.05](http://releases.portswigger.net/2013/02/v1505.html). This release contains a number of feature enhancements and bugfixes, including an extension to the SOCKS proxy support which allows users to specify that all DNS lookups should be done remotely via the proxy. This means that it is possible to test Tor hidden services with Burp.

This feature is currently only available in Burp Pro, but should eventually make its way into the free edition.

### Tor hidden services

[Tor hidden services](https://www.torproject.org/docs/hidden-services.html.en), sometimes also referred to as the *hidden web*, *dark web*, and *deep web*, were deployed on the Tor network in 2004. Hidden services allow users to host various kinds of resources, such as websites and instant messaging servers, without having their identity or location revealed. Not only does this protect the operators, but also the resources.

A hidden service can only be accessed through its Tor-specific *.onion* pseudo-*top-level domain (TLD)*. One example is [3g2upl4pq6kufc4m.onion](http://3g2upl4pq6kufc4m.onion/), which is the address of the [Duck Duck Go](https://duckduckgo.com/) search engine. The relays in the Tor network understand the .onion pseudo-TLD and route data anonymously both to and from the hidden service. The IP address, physical location, or operator of a Tor hidden service is never revealed to relays in the Tor network, visitors, or the Tor Project -- as long as the hidden service has been properly [configured and secured](https://www.torproject.org/docs/tor-hidden-service.html.en).

### Tor hidden services and Burp

To test a Tor hidden service with Burp, make sure the Tor Browser, Tor, and Burp can all talk to each other:

 * Download the [Tor Browser Bundle](https://www.torproject.org/download/download-easy.html.en), [verify the signature](https://www.torproject.org/docs/verifying-signatures.html.en), and get it up and running
 * Figure out which SOCKS port the Tor Browser is using to talk to Tor: *Preferences*, *Advanced*, *Network*, *Settings* (next to *Configure how TorBrowser connects to the Internet*)
 * Configure Burp to use the same SOCKS proxy as the Tor Browser: *Options*, *Connections*, scroll down to the SOCKS proxy section, enter the host and port, and tick *Use SOCKS proxy* and *Do DNS lookups over SOCKS proxy*
 * Configure the Tor Browser to use Burp as an HTTP Proxy: on the proxy settings page, enter *127.0.0.1:8080* as the HTTP proxy and tick *Use this proxy for all server protocols*

Visiting [3g2upl4pq6kufc4m.onion](http://3g2upl4pq6kufc4m.onion/) in the Tor Browser gives the following output in Burp:

{% img /images/burp-tor-hs.png %}
