---
title: About decentralized content-oriented network systems
date: $date$
author:
- <a href="http://www.cds.caltech.edu/~mhucka/">Michael Hucka</a>
---

This project is a work in progress. We are studying network architectures for _content-oriented networks_, that is, networks where the communications protocols center on accessing data by name rather than on sending data packets from a source to a destination address.  The latter is the prevaling paradigm embodied in TCP/IP, which underlies much of the global Internet; the former is an approach originally pioneered in peer-to-peer networks and now under active development by numerous academic and commercial groups.  The most prominent of these is the _named data networking_ (NDN) effort pursued under NSF funding by several research groups, but the general idea of content-addressed networking is broader and includes efforts such as [IPFS](https://ipfs.io), [Blockstack](https://blockstack.org), and others.

We are particularly interested in decentralized and distributed efforts in this area.  By _decentralized_, we mean systems where services and decisions are not concentrated in one or a few powerful entities, and are instead allocated throughout a network.  By _distributed_, we mean systems in which resources (storage, processing, etc.) are distributed across multiple components that communicate with each other.  Decentralization implies distribution, but not all distributed systems are decentralized.

Does it make sense to qualify the term "content-oriented" or "content-addressed" with "decentralized"?  Isn't it redundant?  The position put forward here is that, while networking is inherently about communication between distributed entities (why else would you have a network in the first place?), content addressing per se does not have to be implemented in a decentralized or distributed manner.  In principle, a distributed network _could_ implement content addressing in some kind of centralized way---it would probably not have the most desirable properties, but it is a viable point in the feature-space of architectures.  However, we are interested in understanding the architectures and properties of truly decentralized systems because of their potential to possess highly desirable properties such as robustness against damage and degradation, resilience in the face of adversarial attacks, and more.
