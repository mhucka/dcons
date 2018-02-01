---
title: A synopsis of Named Data Networking (NDN)
date: $date$
author:
- <a href="http://www.cds.caltech.edu/~mhucka/">Michael Hucka</a>
---

# Introduction

The network architecture that underlays today's Internet is largely based around the concept of moving packets of data from one explicitly identified host to another.  Internet Protocol (IP), the predominant networking protocol supporting the Internet, is concerned with addressing hosts and sending data packets between them&mdash;a point-to-point communications approach that made sense in the landscape of users and applications that existed in the 1970's. However, modern uses of the Internet have evolved from their origins, and today, content references and distribution make up the majority of internet traffic.  The best estimate in mid-2017 puts the number of visible web pages at nearly 50&nbsp;billion [@vandenbosch2016estimating], while in terms of data transmitted over the Internet, global IP traffic was estimated at 1.2&nbsp;zettabytes in 2016 and is predicted to reach 3.3 zettabytes per year by 2021, with IP video traffic estimated to comprise 82 percent of that [@cisco2017zettabyte].

"In such a content-centric worldview, what a person wants, rather than where it is located, is what matters most; content, rather than the server on which content resides, becomes the starting point" [@Kurose2014-ur].
The fact that most modern applications are not well matched to IP's host-oriented architecture was already apparent to network architects two decades ago.  This led to efforts by researchers and funding agencies to explore new architectural designs in the early 2000's.  One outcome of this work was the research on Content-Centric Networking [@Stanik2009-yu], which later led to the *Named Data Network* (NDN) project [@Jacobson2009-nt; @Jacobson2012-jc].

NDN changes the focus of network services from delivering packets to a destination (the basic IP approach), to requesting data by name.  Architecturally, NDN puts named data chunks at the "thin waist" of the network protocol stack, replacing host-addressed IP packets.  (See the next figure.)  The name in an NDN packet can refer to anything: a file, a data segment from a movie, a data stream endpoint from an environmental sensor, a command to control something, etc.  Data chunks themselves are signed digitally, with a certification scheme that allows a consumer to verify that the content was produced by the expected source and that the data has not been tampered with.

<p align="center"><!-- need this p element to center figure on GitHub -->
<figure align="center">
  <img style="zoom: 75%" src="ndn-hourglass.svg">
  <figcaption>No discussion of Internet protocols is complete without the canonical hourglass diagram of the network stack.  (Left) Today's IP-based Internet architecture. (Right) The NDN-based network architecture.  The "thin waist" part of NDN consists of data chunks; this replaces the Internet Protocol packets in the regular IP stack. (Figure based on diagram by Jacobsen et al., 2012.)
</figcaption>
</figure>
<p>

During normal operation, a user application generates requests for content and these requests are forwarded between NDN routers in a network until they reach the relevant content source(s).  The nature of how data is packaged in NDN (discussed below) allows content can be cached or delivered from anywhere else, too.  The provision for the network infrastructure to cache content as a by-product of its operation offers the potential for content to be found and reused closer to user destinations, thereby reducing network load and response latency.  The nature of NDN data also also the content *itself* to be secured&mdash;not by securing the communication channel (which is how IP does it) but by securing the chunks of data.  This is fundamentally a better match to the needs of today's Internet applications.

# General system components and operation

NDN communication is initiated when a consumer makes a request for data. In the NDN protocol, a request for data is expressed by broadcasting an *Interest Packet* on a network interface connected to an NDN network.  Any node in the network having data that satisfies the Interest can respond with a *Data Packet*.  A Data Packet is only transmitted in response to an Interest Packet, and it consumes that Interest, thereby maintaining a flow balance.

Interest Packets primarily consist of a name and optional selectors to specify more precisely the data being requested; Data Packets consist of a name, some metadata, digital signature data, and the content requested [@Jacobson2012-jc].  As discussed further below, names in NDN are typically structured hierarchically.  A Data Packet "satisfies" an Interest if the name in the Interest is a prefix of the Name in the Data Packet; in other words, if the Data is in the subtree implied by the name in the Interest.

<p align="center"><!-- need this p element to center figure on GitHub -->
<figure align="center">
  <img style="zoom: 75%" src="ndn-packets.svg">
  <figcaption style="text-align: center">Illustration of packets in NDN.
</figcaption>
</figure>
<p>

Routers forward Interest Packets to producers that can potentially answer the requests.  In the general case, a producer sends back to the consumer by retracing the path through the network in reverse; however, caching may alter this path in practice.

## NDN router architecture

To support the operation of NDN networks, there are three main architectural elements in NDN routers: a content store, a pending interest table, and a forwarding information base.

* The *Content Store* (CS): An Data Packet in NDN is idempotent, self-identifying, and self-authenticating.  This means that any given data chunk can be cached and served to multiple consumers.  The content store in an NDN router is used to cache data temporarily that the router sees in response to Interest Packets, so that if the same Interest Packet is requested again in the future by other consumers, the router can provide the data directly without routing the request to the source again.  This increases data sharing, reduces retrieval time and reduces bandwidth consumption.

* The *Pending Interest Table* (PIT): To track requests for data that have not yet been satisfied, an NDN router maintains an entry for each incoming Interest Packet until its corresponding Data Packet arrives or the time-to-live parameter on the request expires.  In NDN, only Interest Packets are routed, and this tracking of packets in effect leaves a trail of breadcrumbs from requester to producer.  The PIT entries are used to forward Data Packets back to consumers, thus following the breadcrumbs.  PIT entries are erased after they are used to forward a matching Data Packet.

* The *Forwarding Information Base* (FIB): An NDN router tracks reachable destinations and the next hops in the network using an FIB. The contents of the FIB are created according to a routing protocol used by an NDN network.  Unlike the equivalent in IP routing, an FIB entry has a list of outgoing interfaces rather than a single one because in NDN, multiple sources can be queried for data simultaneously.

<!--
A design principle in NDN is stateful forwarding, whereby NDN routers keep state information about recently-forwarded packets; this allows such things as flow balance, loop detection, caching, smart forwarding, and more.
-->

The following diagram illustrates the basic processes that take place in an NDN router in response to Interest Packets and Data Packets. The top half shows an Interest Packet arriving at a router. If the router finds an entry for the named data item in its CS, it returns the data on the interface through which the Interest Packet arrived. Otherwise, it next checks the PIT to see if something else has already requested the same data. If it finds a match in the PIT, it adds the interest of the new request to the list of requesters in the PIT entry. When a Data Packet matching an Interest Packet eventually arrives in response to the request, it is forwarded to multiple requesters based on the list in the PIT (a process known as interest aggregation). Finally, if no PIT entry is found, the Interest Packet is passed to the FIB, which performs longest-prefix match (LPM) on the name to look for a network interface that can answer the request. The router forwards the Interest Request to the next hop based on the most specific match in the FIB, and also creates a new PIT entry to identify the router interface on which the interest is pending. If no suitable match is found in the FIB, then the Interest Packet is flooded to all the outgoing interfaces or else deleted (depending on the router's policies).

<p align="center"><!-- need this p element to center figure on GitHub -->
<figure align="center">
  <img style="zoom: 75%" src="main-ndn-router-components.svg">
  <figcaption>Illustration of the main components of an NDN router. The top half illustrates the actions when an Interest Packet reaches the router; the bottom half represents the actions when a Data Packet arrives.  (Diagram based on figure by Saxena et al., which in turn is based on figures from NDN project reports [@Saxena2016-nf; @Zhang2014-zn].)
  </figcaption>
</figure>
<p>

The bottom half of the diagram shows the process when a Data Packet arrives at the router.  The process is simpler because Data Packets are not routed in NDN networks---they only follow the chain of PIT entries back to requester(s).  Upon an incoming Data Packet, a router first searches its PIT for entries that requested the same data item.  If a match is found, the data is passed to all interfaces mentioned in the list, and (depending on the router's caching policy) also cached in the CS.  If no matching PIT entry is found, it means the data was unsolicited and the Data Packet is dropped.

## Data naming in NDN

Interest and Data Packets do not carry any host or interface addresses.  A "name" in NDN is a sequence of bit strings.  The interpretation of names is application-dependent, although the components are usually structured hierarchically and often interpretable as character strings such as "/edu.institution/something/somethingelse".  Hierarchical names of the form "/x/y/z" resemble web URLs, and so are likely to be familiar and convenient for human users to work with; however, the names are opaque to NDN, and delimiters are not part of the names and not included in packet encodings. All that NDN routers do is pay attention to is the structure of names.  Name components may even be encrypted.

The components in names are used to perform hierarchical prefix matching. The first or highest-level component of names are globally-routable names (i.e., namespaces).  These may each be controlled by separate entities, or multiple namespace prefixes may be controlled by the same entity, and the entities may be distributed on a global network or they may be local (e.g., local servers, local sensors that produce data, etc.) Globally-available data must have globally-unique names, but local communications involving local routing can rely on local names.

Routers forward Interest Packets to data producers based on the name of the data in the packet, and Data Packets are returned based on the PIT information in routers established when the packet was forwarded from router to router to (eventual) producer.  Generally, one Interest Packet results in one Data Packet, but fetching large objects that may be divided into multiple Data Packets requires other approaches.  One option is for consumers and producers to agree on a dynamic naming scheme that can be used by applications to allow consumers to construct names deterministically without having seen the entire data ahead of time. Another option is to use interest selectors in Interest Packets. The NDN packet format defines a number of operators that can be used in interest selectors to specify wildcard matches against data name suffixes and/or child elements, and this can be used to request multipart data such as might happen when requesting video streams.


## Caching

An important aspect of NDN operation are the processes of caching data at network nodes.  Implementing cache support in NDN requires the solution of two main problems [@Saxena2016-nf], cache placement and cache update, both of which continue to be active research areas today.  As its name implies, the topic of _cache placement_ is concerned with storing copies of data: where to cache them in the network, how many copies to cache, how to coordinate caching policies between routers, and so on.  Conversely, _cache update_ is concerned with the procedures implemented by routers for updating the content held in caches.  Issues that enter into the equation include the popularity of any given piece of content.

Several benefits accrue from caching.  First, content can be uncoupled from producers, so that producers do not have to be accessed (or even accessible) every time a consumer asks for particular content---potentially eliminating a single point of failure.  Caching performed by a node in the network can help cope with intermittent network connectivity; mobile nodes can act as content routers between physically disconnected networks.  Second, it reduces load on the producer by making content available from multiple sources (i.e., not just the producer but cache sources).  Third, if managed properly, it can support more efficient multicasting to many consumers.  And fourth, caching can reduce network load and network latency [@Saxena2016-nf; ].

## Forwarding

As mentioned above, the operation of an NDN network involves queries by consumers, in the form of Interest Packets being passed around the network until they reach a consumer, who (in the generic case) respond with a Data Packet that is passed back to the consumer.  This operation requires network routers to support efficient interpretation of request and data packets and manage PIT and FIB tables.  Since routers need to process a very high number of network transactions per second, the development of approaches for scalable forwarding has been an important topic in NDN development.

The challenge in forwarding packets lies in NDN's use of unbounded names rather than fixed-length host addresses.  This means that routers need to perform character-based prefix matching; at the same time, they also need to manage millions or billions of entries in their tables in order to route network traffic.  Operations need to be performed with low latencies, which requires the fastest memory accesses possible, but processors typically have limited amounts of fast on-chip RAM.  Thus, operation and management of the PIT table can become a bottleneck.  A variety of approaches are being pursued to solve this problem [@Saxena2016-nf].

A separate concern in implementing NDN routers is the FIB table used for Interest Packet forwarding.  Once again, the data structures and hardware used to implement the FIB need to be efficient and scale to a large number of constantly-changing entries.  This has led to work on both strategies and hardware implementations.  On the strategy front, implementing efficient forwarding approaches requires addressing issues of discovering and selecting paths for Interest Packets, congestion control, and more.  On the hardware front, work in this area has focused not only on CPU-based approaches but also GPU-based and FPGA-based solutions.


# Security and privacy

The architecture of IP was not designed with an eye towards secure data distribution.  NDN is fundamentally different and directly implements content-based security: "protection and trust are properties of the content itself, not of the connections over which it travels" [@Jacobson2012-jc].  In NDN, every datagram is cryptographically authenticated.  This not only avoids some of the existing issues of host-based trust and security; it means that NDN provides end-to-end security between content producer and consumer, and data in NDN is _secure at rest, not just during transmission_.

## Data security

Every Data Packet in NDN is digitally signed using public key cryptography, allowing a receiver to verify the authenticity of the data. The NDN data packet format includes a field for the signature, and a field called the key locator. The key locator is the name (in the NDN sense) of the public key that was used to create the signature. In NDN, a key is another packet of named NDN content, and like all NDN data, it is signed too, to verify it. A consumer can follow the chain of keys to a root key to authenticate any Data Packet. This arrangement provides authentication (a recipient can verify the content was created by a known sender) and data integrity (a recipient can be sure the content has not been modified). The digital signing of packets not only means that data is secured while it is in transmission: since NDN Data Packets are also units of storage, the scheme protects data at rest. Cached data and stored data are secured as a consequence.

Producers can chose the signature algorithm.  Verification of signatures may involve multiple rounds of fetching and decryption to get to follow the chain of signatures to a recognized signature; a way to cope with the inherent performance issues is for an NDN-based system to cache validated certificates for some amount of time.

The content of Data Packets is up to applications, and the contents could also be encrypted in addition to the Data Packet itself being signed.

## Network security

Some types of network attacks that are common against today's network infrastructure are mitigated or eliminated by the virtue of how NDN works. For example, port scans of given hosts to find running services is not feasible in NDN: there are no hosts in the same sense as there are in TCP/IP.  The equivalent attack would require some kind of "name scan" in order to guess what services might be provided under a given namespace, but since there can be an essentially infinite number of names, the attack is impractical.

Some types of attacks are still possible. For example, bandwidth depletion attacks involve flooding a victim with requests for content. This is a potential attack approach in NDN, but it is mitigated by the fact that once retrieved, content will be cached by intermediate routers between an attacker and providers for a target namespace.

Some new types of attacks are introduced by NDN. One example is *interest flooding*, in which attackers unleash a high number of unsatisfiable Interest Packets, causing routers to process a large number of packets (including potentially filling their PIT with fake requests). Another potential attack is *content cache poisoning*, in which an attacker seeks to overload router caches with corrupted Data Packets possessing fake signatures. While fakes can be detected, the overhead of processing them may impact router and network performance. These new types of attacks are under study by NDN developers.

## Privacy of communications

A final area of development concerns privacy of communications. The nature of the threat is different from current host-based networking: data itself can be secured as described above, and since NDN does not have destination hosts, the question of who is consuming what content is much more difficult to answer.  However, certain kinds of analysis could still compromise privacy. An example is that the names of data being requested could perhaps be used via some kind of correlation analysis to infer producers and/or consumers.


<!--
# Consequences of NDN's architecture

The resulting system overcomes a number of limitations caused by the use of addresses in IP.  First, network address space exhaustion is not an issue---NDN namespaces are unlimited, unlike IPv4 addresses.  Second, network address translation (NAT) is no longer necessary when crossing different subnetworks or domains: since there are no addresses in NDN (public or private), there is no need for address translation.  Third, address assignment and management in local networks is no longer necessary.

challenges...

-->


# Additional reading

The following list of materials, in the order given here, may help readers become more familiar with NDN.

1. Kurose's paper focuses on cache modeling, but the text through section 2.3 provides a nice, high-level summary of past network architectures and the architecture implied by NDN [@Kurose2014-ur].  
2. Jacobson et al.'s 2012 CACM paper is slightly out of date with respect to the state of NDN today, but it is a good and concise explanation of all the essential elements of NDN [@Jacobson2012-jc].
3. Saxena et al.'s 2016 paper is the most comprehensive survey of work in NDN so far [@Saxena2016-nf].  It gathers into one place a summary and references to work on all aspects of NDN, and provides helpful taxonomies of different areas of work.  (Note that the abstract may leave a poor impression, but the rest of the paper is considerably better.)
4. The paper by Yu et al. explains in much better detail how security and trust are expected to work in NDN [@Yu2015-gz].


# References
