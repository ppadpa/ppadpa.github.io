---
layout: post
title: "advnaced DNS learning notes"
date: 2020-03-25
---

# RFC guidance
### Rfc background
IETF: Internet Engineering Task Force - a group, want to “make the Internet work better”

|RFC (request for comments) category| |
| ----------- | ----------- |
|Standards track | proposed standard, internet standard document |
|BCP (best current practice) |anything, not standards|
|Information| not standards||
|Experimental| not recommended for use|
|Historical| |

&nbsp;
# part1: basic concepts - standards
> RFC1034/1035: internet standards
- Domain Namespace

<p align="center">
  <img width="460" height="300" src="{{site.baseurl}}/assets/path/1.jpg">
</p>

- DNS query flow

![NDS_flow]({{site.baseurl}}/assets/path/2.jpg "NDS_flow"){:class="img-responsive"}

- Resource record (RR), many categories

  -A
  
  -AAAA
  
  -NS
  
  -MX
  
  -CNAME
  
  -SOA
  
  -......
  
- Port 53 is normally used
![record format]({{site.baseurl}}/assets/path/3.jpg "format"){:class="img-responsive"}

&nbsp;
### DNS query
![NDS_query format]({{site.baseurl}}/assets/path/4.jpg "query format details"){:class="img-responsive"}

&nbsp;
### TCP or UDP?

- Indicated by the TC flag

- Single UDP message is limited to 512 bytes, while TCP doesn’t have limit

- IfTCflag=1,

  - indicate the message was truncated due to its length being longer than the maximum limit.
  
![NDS_query format]({{site.baseurl}}/assets/path/5.jpg "query format details"){:class="img-responsive"}

&nbsp;
### EDNS(0) – extension mechanism for DNS

>RFC6891: internet standards

- Add to additional information section, for backward capability.

- Introduce a new RR type called “OPT RR”

![EDNS]({{site.baseurl}}/assets/path/6.jpg "EDNS0 format"){:class="img-responsive"}

- Add to additional information section, for backward capability

- Introduce a new RR type called “OPT RR”

- RCODE – previously 4 bit, now extra 8 bit in additional data
  
  - Support more response code

- Sender’s UDP payload size
  
  - Largest UDP payload that can be reassembled and delivered in requester’s network stack
  
  - If less than 512, treat as 512

![EDNS]({{site.baseurl}}/assets/path/7.jpg "EDNS0 format"){:class="img-responsive"}

&nbsp;
### DoT (DNS over TLS)

> RFC7858, 8310: proposed standards

- Port 53 is not allowed, by default, server listen to port 853.

- DoHandshake
  
  - Client hello: specify TLS
  
  - Server hello: TLS parameters, certificate message (with id, digital signature)
  
  - Client check cert, by public pinning

- DNS message will then follow TLS format

![DOT]({{site.baseurl}}/assets/path/8.jpg "DOT"){:class="img-responsive"}

&nbsp;
### DoH (DNS over HTTPs)

> RFC 8484: proposed standards

- Just like normal HTTPs, use TCP port 443

- TLS 1.2, 1.3 supported, based on Name server

- Query either with HTTP GET/POST

![DOH]({{site.baseurl}}/assets/path/9.jpg "DOH"){:class="img-responsive"}

&nbsp;
#### DoT vs DoH, and their common drawbacks

- DoT will easily cause firewall block, because of it’s special port number

- DoH follows HTTPs protocol, and if TLS or HTTP upgraded, it will also follow the design, enjoy further security protection -

> there is http/3 (QUIC), believed to improve HTTP performance and security a lot.

- limit of servers support this service.

- Increase RTT

&nbsp;
# part2: use cases & discussion on DNS

### UDP or TCP?

- For normal DNS, client SHOULD resend post with TCP if the response TC bit is set to 1.
  
  - When response is too large (too many number of records), server will choose to fall back to TCP
  
  - For name servers, resolvers, zone data transfer might use TCP.
  
  - RRL (response rate limiting), to prevent reflection attack &UDP amplification. a limit is set if one client seems keeps sending request. Server will terminate UDP, and send back response with TC bit = 1.

- For DoH & DoT, TCP is used. However, if server doesn’t support these 2 modes, query will fall back to unencrypted DNS.

&nbsp;
### Security considerations
One famous use case of EDNS is called DNSSEC
  
  - to prevent MITM, DNSSpoof, cache poisoning
  
  - Basically, resolver have signature of server’s zone records to ensure integrity

&nbsp;
### DNSSEC

> RFC 5011, 6840, 4033, 4044 and many other DNSSEC specifications/clarifications (internet standard, proposed standards)

![DNSSEC]({{site.baseurl}}/assets/path/10.jpg "DNSSEC"){:class="img-responsive"}

&nbsp;
### DNSSEC & DoT/DoH & TLS1.3

- DNSSEC & DoT/DoH: complementary and both ensure security. • DNSSEC: authentivity & integrity, not confidentiality

- DoT/DoH: just as TLS. But it can’t avoid poisoning

- In one proposal, TLS1.3 should use encrypted SNI

- To get server’s public key, DoT/DoH is used.

- Resolver use DNSSEC to ensure the above DoT/DoH is trustable • In case of DNS poison

> Experimental: Encrypted Server Name Indication for TLS 1.3; Already supported by Firefox Nightly

&nbsp;
### DANE (DNS-based Authentication of Named Entities)

> Rfc 6698 proposed standards

> How to generate and verify DANE: https://blog.apnic.net/2017/01/06/lets-encrypt-dane/

- TLS/SSL use CA, but CA suffer from security breaches

- DANE ask domain admin store key in DNS (TLSA RR)

- use DNSSEC to verify

&nbsp;
### EDNS other use case

> proposed standards

- Rfc 7830 set how much padding should be around a DNS message

- Prevent property leaking based on message length
  
> proposed standards

-  Rfc 7828 The edns-tcp-keepalive EDNS0 Option
  
  - indicating how long a TCP connection should be kept alive
  
  - Server should only accept TCP query with this extension
  
> Informational, but used a lot

- Rfc 7871 client subnet in DNS query, for CDN
  
  - Indicate sender’s geographic locations

&nbsp;
# Part3: other fun industry use case

`DNS is just for service discovery`

## Spotify uses DNS for song lookup (not cached)

![Spotify1]({{site.baseurl}}/assets/path/11.jpg "Spotify1"){:class="img-responsive"}

Dns query: txt record of key. And txt record contains information of the server that save the song.

> spotify: https://labs.spotify.com/2017/03/31/spotifys-lovehate-relationship-with-dns/

&nbsp;
## Microservice discovery

Because widespread adoption of DNS infrastructure

![Spotify2]({{site.baseurl}}/assets/path/12.jpg "Spotify2"){:class="img-responsive"}

&nbsp;
# Part4: implementations

## Unix environment

- Divison1: Linux, a free unix-like operating system
  
  - Linux:writtenbyLinuxTorvald

- Divison2: System V release.
  
  - one derivative is Solaries 10, developed and maintained by SUN (Oracle).
  
  - Most other system V derivatives are replaced by Linux.

- Division3: BSD
  
  - BSD “Berkeley software distribution”
  
  - Origins from Berkeley’s computer system research group

- notable: Apple Mac OS X (Darwin is based on FreeBSD)
  
  - FreeBSD: one of BSD operating systems. a descendant of 4.4 BSD release

&nbsp;
## DNSSEC, DoT and DoH support

-  DNS service provider
  
  - DNSSEC: widely support
  
  - DoT & DoH: at least Cloudflare’s 1.1.1.1 and Google’s 8.8.8.8 support it

- Client side:
  
  - Linux: /etc/systemd/resolved.conf — Network Name Resolution configuration files
    
    - DNSOverTLS flags
    
    - edns0
  
  - FreeBSD: BIND resolver is used
   
    - var/run/resolv.conf (alias /etc/resolv.conf) – DNS configuration
    
&nbsp;
Thank for the support from Junyi and my family.

powered by [Jekyll](http://jekyllrb.com). Markdown is used
