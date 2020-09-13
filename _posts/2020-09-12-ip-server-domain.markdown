---
layout: post
title:  "Domains, Servers, and IPs (aka no, that's not a subdomain takeover)"
date:   2020-09-12 12:00:00 -0700
categories: sto ip domain bugbounty
---

# Domains, Servers, and IPs (aka no, that's not a subdomain takeover)

Over the past couple months I've seen a lot of confused questions about subdomain takeovers (STO). Some (paraphrased) highlights:
- "I just got a 500, does this mean I can get subdomain takeover?"
- "The domain has no IP, STO?"
- "I'm seeing a default apache page, STO"?

Now, I get why the mind goes to STO. You're doing domain recon, you think you're seeing something that no one has dug up before and you want a $1k+ payout. I'm writing this blog post because the questions (and follow up conversations) reveal that some hunters don't actually understand the mechanics of either the internet or STO.

**Disclaimer** This post is reasonably general and exceptions could/do exist. I don't intend on calling out every possibility that could lead to given behaviour.

## Definitions

Yeah, these are important. If you use domain/server/IP interchangeably you're missing important information. These definitions are not the RFC definitions or really precise at all.

### Domain

A domain is a string that corresponds to one or more records on a DNS server. That's it. If you own a domain you control the DNS records of the domain and it's subdomains.

There are different types of records that mean different things (citation needed) but for STO the most important two are the `A` record and the `CNAME` record. `A` records are IP addresses. `CNAME` records are other domains and are effectively an alias to that other domain (but different [from "ALIAS" records](https://support.dnsimple.com/articles/differences-between-a-cname-alias-url/)).

### IP

This is an Internet Protocol address. You can (attempt) to send packets to any IP you want. Whether it gets there are not depends on a lot of things, and how/if it responds is up to whatever is at that address.

DNS `A` records are IP addresses. This tells your computer which IP to send HTTP requests to.

### (HTTP) Server

An HTTP server is a process that responds to HTTP requests and is usually listening on TCP ports 80 or 443.

## Subdomain Takeover

Subdomain takeover is when an attacker is able perform an action so that an HTTP request to `${vulernablesubdomain}.victim.com` goes to an attacker controlled server.

This usually happens in two ways:
- bad `A` records
- bad `CNAME` records

### Bad `A` Records

This is when the domain owner has set the `A` record of a subdomain to an IP that they do not control. If the attacker is able to gain control of this IP, then you've got subdomain takeover. Keep in mind that "IP they do not control" and "IP that you (the attacker) cannot access" can behave in similar ways. Keep this in mind and do not report a subdomain takeover without a PoC unless you want to lose rep and signal.

### Bad `CNAME` records

This is when the domain owner has set the `CNAME` record to point to a 3rd party domain which they no longer control. The most common example of this is with AWS S3 buckets - if the `CNAME` points to a bucket and that bucket does not exist, then you've got an easy subdomain takeover. The main tool used for this is the excellent [Can I Takeover XYZ?](https://github.com/edoverflow/can-i-take-over-xyz) by EdOverflow.

## Verifying STO

Now, let's figure out whether or not you've found an STO. For this I'm going to assume that you're doing subdomain recon and you've found some odd behaviour that you want to investigate.

1. Look at the DNS records for the subdomain. If it is using an `A` record and you are connecting to an HTTP server then you do not have subdomain takeover. (Actually that's not true because there's a chance that you're making it to a multihosting server that is giving a default response for hosts it doesn't recognize, and then you could have STO, but this is relatively rare)
2. If it's an `A` record and you are not connecting to an HTTP server, figure out what you can about the IP. There's a small chance that the IP is in the range of a hosting provider like AWS, and you can get control of the IP. This is a long shot because most hosting providers do not allow you to request specific IPs. Keep in mind the private IP ranges [RFC 1918](https://tools.ietf.org/html/rfc1918) since these IPs will be unreachable and do not indicate an STO vulnerability.
3. If the domain is using a `CNAME` record, check whether it's pointed to an [exploitable 3rd party](https://github.com/edoverflow/can-i-take-over-xyz) and is setup correctly. If the 3rd party is not exploitable or things are setup correctly, you do not have an STO.

## Conclusion

STO is caused by a specific type of DNS technical debt. A DNS record was setup that is no longer valid, and the record has not been deleted. If you find something that cannot be explained that way, it is not an STO.
