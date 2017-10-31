About decentralized content-oriented networking systems
=======================================================

This is the source code repo for our [online notebook](https://mhucka.github.io/dcons) summarizing some research into network architectures that are variously known as _information-centric networking_, _content-centric networking_, _content-addressed networks_, and _named data networking_.

*Author*:      [Michael Hucka](http://github.com/mhucka) and [John C. Doyle](http://www.cds.caltech.edu/~doyle)<br>
*Repository*:   [https://github.com/mhucka/dcons](https://github.com/mhucka/dcons)<br>
*Pages*:       [https://mhucka.github.io/dcons](https://mhucka.github.io/dcons)<br>
*License*:     [Creative Commons Attribution License 4.0](https://creativecommons.org/licenses/by/4.0/legalcode)

☀ Introduction
-----------------------------

We are studying network architectures for _content-oriented networks_, that is, networks where the communications protocols center on accessing data by name rather than on sending data packets from a source to a destination address.  The latter is the prevaling paradigm embodied in TCP/IP, which underlies much of the global Internet; the former is an approach originally pioneered in peer-to-peer networks and now under active development by numerous academic and commercial groups.  The most prominent of these is the [_named data networking_ (NDN)](http://named-data.net) effort pursued under NSF funding by several research groups, but the general idea of content-addressed networking is broader and includes efforts such as [IPFS](https://ipfs.io), [Blockstack](https://blockstack.org), and others.

We are particularly interested in decentralized and distributed efforts in this area.  By _decentralized_, we mean systems where services and decisions are not concentrated in one or a few powerful entities, and are instead allocated throughout a network.  (The [Wikipedia article about decentralization](https://en.wikipedia.org/wiki/Decentralization) does a decent job of discussing the broader senses of the term both inside and outside of technological contexts.)  By _distributed_, we mean systems in which resources (storage, processing, etc.) are distributed across multiple components that communicate with each other.  Decentralization implies distribution, but not all distributed systems are decentralized.  We are interested in understanding the architectures and properties of decentralized systems because of their potential to possess highly desirable properties such as robustness against damage and degradation, resilience in the face of adversarial attacks, and more.


⁇ Getting help and support
--------------------------

If you find an issue, please submit it in [the GitHub issue tracker](https://github.com/mhucka/dcons/issues) for this repository.
