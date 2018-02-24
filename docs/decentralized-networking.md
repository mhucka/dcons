---
title: About decentralized content-oriented network systems
date: $date$
author:
- <a href="http://www.cds.caltech.edu/~mhucka/">Michael Hucka</a>
---

This project is a work in progress. We are studying network architectures for _content-oriented networks_, that is, networks where the communications protocols center on accessing data by name or content rather than by location.  The latter is the prevailing paradigm embodied in TCP/IP, which underlies much of the global Internet and is based on pair-wise communication between identified hosts.  The idea of focusing on content instead of hosts as an architectural foundation for the Internet first gained significant attention in the 1990's, thanks in part to the DARPA Next-Generation Internet program that funded efforts such as [TRIAD](https://web.archive.org/web/20010409204855/http://dsg.stanford.edu/triad/index.html), as well as research into decentralized peer-to-peer content storage systems such as [Oceanstore](http://oceanstore.cs.berkeley.edu/).  In truth, the principles are older and go back at least to the 1970's, but the explosive growth of the World Wide Web in the late 1990's led many people to notice that most Internet traffic was HTTP-based and consisted of content such as images and videos, a scenario for which the existing Internet infrastructure was suboptimal.  Efforts to design a new Internet architecture focusing on _Information-Centric Networking_ (ICN) soon arose, with one specific research strand in the USA being known as _Content-Centric Networking_ (CCN) and later [_Named Data Networking_ (NDN)](http://named-data.net).  The latter is a prominent NSF-funded research project in the USA today involving many institutions, but activity on other content-centric networking approaches continues worldwide, especially in Europe.

Producing a new Internet architecture is one (and arguably the preferred) way of addressing the need for working with content more efficiently, but absent an overhaul of Internet infrastructure, it is also possible to achieve similar effects by layering content-oriented functionality over existing networks.  This is the tack taken by numerous recent projects that leverage peer-to-peer network approaches, including [IPFS](https://ipfs.io), [Blockstack](https://blockstack.org), [MaidSafe](https://maidsafe.net), and others.  These also often focus on additional features such as privacy and data persistence.

We are particularly interested in decentralized and distributed network architectures.  By _decentralized_, we mean systems where services and decisions are not concentrated in one or a few powerful entities, and are instead allocated throughout a network.  By _distributed_, we mean systems in which resources (storage, processing, etc.) are distributed across multiple components that communicate with each other.  Decentralization implies distribution, but not all distributed systems are decentralized.  We are interested in understanding the architectures and properties of truly decentralized systems because of their potential to possess highly desirable properties such as robustness against damage and degradation, resilience in the face of adversarial attacks, and more.  Between the many approaches being pursued today, there are certainly tradeoffs, and a future ICN/CCN/NDN-based Internet may even be able to replace much of the underlying functionality that systems such as IPFS today have to build up explicitly.  Still, not all of the tradeoffs or implications are obvious at this time.

Many technical problems will need to be solved before content-oriented networks become a significant part of network infrastructure in the world.  Additional technical challenges enter into the picture when mobile and transiently-connected elements are involved, which has led to many active areas of work today in such things as [mobile ad hoc networks](https://en.wikipedia.org/wiki/Mobile_ad_hoc_network) (MANET) and [delay tolerant networks](https://en.wikipedia.org/wiki/Delay-tolerant_networking) (DTN).  How are these related to decentralized content-oriented networks?  Can current efforts such as NDN and IPFS support mobility and delays?

Those are the kinds of questions we hope to answer!