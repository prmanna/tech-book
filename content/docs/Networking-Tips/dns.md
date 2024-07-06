---
title: "DNS Overview"
bookCollapseSection: true
weight: 10
---

# Intro
DNS (Domain Name System) allows you to interact with devices on the Internet without having to remember long strings of numbers. 

## What is the Need for DNS?
Every host is identified by the IP address but remembering numbers is very difficult for people also the IP addresses are not static therefore a mapping is required to change the domain name to the IP address. So DNS is used to convert the domain name of the websites to their numerical IP address. 

## Types of Domain
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/dns/Types-of-DNS.png)

## Domain Hierarchy

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/dns/domain-660x380.png)
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/dns/DNS-extra.png)

TLD (Top-Level Domain) is the rightmost part of a domain name. The TLD for geeksforgeeks.com is “.com”. TLDs are divided into two categories: gTLDs (generic top-level domains) and ccTLDs (country code top-level domains). Historically, the purpose of a common top-level domain (gTLD) was to inform users of the purpose of the domain name; For example, a.com would be for business purposes, .org for organization, .edu for education, and .gov for the government. And a country code top-level domain (ccTLD) was used for geographic purposes, such as .ca for Canadian sites, .co.uk for UK sites, and so on. As a result of the high demand, many new gTLDs have emerged, including.online,.club,.website,.biz, and many others.

SLD(Second-Level Domain): The .org component of geeksforgeeks.org is the top-level domain, while geeksforgeeks is the second-level domain. Second-level domains can only contain a-z 0-9 and hyphens and are limited to 63 characters and TLDs when registering a domain name (may not start or end with hyphens or contain consecutive hyphens).

Subdomain: A period is used to separate a subdomain from a second-level domain. For example, the admin part is a subdomain named admin.geeksforgeeks.org. A subdomain name, like a second-level domain, is restricted to 63 characters and can only contain the letters a-z, 0-9, and hyphens (cannot begin or end with hyphens or consecutive hyphens).To create longer names, you can use multiple subdomains separated by periods, such as mailer.servers.geeksforgeeks.org. However, the maximum length should not exceed 253 characters. You can create as many subdomains as you want for your domain name.

DNS Record Types: However, DNS is not just for websites, and there are many other types of DNS records as well. We’ll go through some of the most common ones you’re likely to encounter.

* A Record – For example, 104.26.10.228 is an IPv4 address that these entries resolve to.
* AAAA Record – For example, 2506:4700:20::681a:bc6 resolves to an IPv6 address.
* CNAME Record – For example, the subdomain name of Geeksforgeeks’s online shop is marketing.geeksforgeeks.org, which gives a CNAME record of marketing.shopify.com. To determine the IP address, another DNS request will be sent to marketing.shopify.com.
* MX Record – These records point to the servers that handle the email for the domain you are looking for. For example, the MX record response for geeksforgeeks.com would look like alt1.aspmx.l.google.com. There is also a priority sign on these documents. It instructs the client in which order to try the servers. This is useful when the primary server fails and the email needs to be sent to a backup server.
* TXT Record – TXT records are text fields that can be used to store any text-based data. TXT records can be used for a variety of things, but one of the most common is to identify the server that has the authorization to send an email on behalf of the domain (this can help in the fight against spam and fake email). is). They can also be used to verify domain ownership when registering for third-party services.

## How Does DNS Work?
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/dns/How-DNS-Works-gif.gif)

## What Are The Steps in a DNS Lookup?
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/dns/Working-of-DNS.gif)

**References:**
* [DNS](https://www.geeksforgeeks.org/domain-name-system-dns-in-application-layer/)
