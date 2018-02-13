---
title: A synopsis of the InterPlanetary File System (IPFS)
date: $date$
author:
- <a href="http://www.cds.caltech.edu/~mhucka/">Michael Hucka</a>
---

# Introduction

IPFS aims to solve several problems at once: efficient and secure decentralized storage, content-oriented addressing instead of location-based addressing, and immutable data.  IPFS has some superficial similarities to NDN [@Jacobson2012-jc; @Saxena2016-nf], especially in the way that both aim to change the host-based addressing and point-to-point communications approaches of today's internet; however, they are markedly different in their approaches.

The IPFS project begain in 2014, initially designed by software developer Juan Benet [@Benet2014-sp] and now being developed by a commercial venture, [Protocol Labs](http://ipfs.io) as an open-source project.  It bills itself as the "Distributed Web", reflecting its emphasis on replacing current web protocols with a content-addressed, distributed storage scheme.  Much like NDN, IPFS changes the focus of network services from delivering packets to a destination (the basic approach with TCP/IP), to requesting content by an identifier.  To achieve its goals, IPFS amalgamates several technologies.  Storage is achieved by distributing content across a network of peers; distribution is achieved using a peer-to-peer protocol derived from BitTorrent.  Content is "named" by the value computed by a hash function _from the content itself_; this hash value is a unique fingerprint for every block or sequence of bytes.  (This is in contrast to NDN, which uses separate, hierarchically-structured names.)  The system is fully decentralized, with no central authority.  It does not require DNS: content is resolved over the peer-to-peer network using a distributed hash table and does not involve DNS name lookups.  Nor does IPFS require host authority certificates.

Like NDN, IPFS promises to lighten network loads by caching copies of content around a network; retrieval can thus potentially use the node closest to a client, rather than from a specific server.  Unlike NDN, IPFS does not seek to change the Internet architecture at the level of packet protocols and network routing.  In fact, current IPFS implementations leverage existing Internet protocols to implement an overlay network, and so IPFS can exist on top of many different underlying networking protocols.

IPFS is not the only effort of its type today.  Dat [@Ogden2017-te], MaidSafe [@Lomas2016-zz], Storj [@Wilkinson2014-kj], Blockstack [@Ali2016-ga] and Freenet [@Clarke2002-sc] are probably the closest related systems.  In all cases, these projects aim to provide decentralized content storage that is addressed by keys rather than traditional locations, and is resistant to censorship and network fragmentation.  Each participanting node's storage and bandwidth are harnessed to provide, collectively, a large distributed storage network.  Some of the efforts such as Freenet emphasize anonymity and security as a philosophical and technical stance; others such as IPFS emphasize the potential for permanence of content.


# General system components and operation

The basic function of IPFS is to provide client applications with an API that is oriented towards storing and retrieving content identified by alphanumeric text strings.  These text strings are computed directly from the content itself: given a block of data, a mathematical formula known as a _cryptographic hash function_ is used to produce an identifier string that is unique to that block of data, such that a change of even one bit in the data will lead to a different value.  The value is known as a _hash_ in IPFS parlance.  If we think of IPFS as a large key-value storage system, then a hash is used as a key and content is stored as a value associated with that key, but with a twist: the key is derived from the content itself.

Deriving an identifier for a block of data in this way is pivotal to several benefits offered by IPFS.  The first is natural deduplication: given that any given block of data is uniquely identified by its hash value, and no two blocks of data will have the same hash unless they are identical, it follows that IPFS only needs to store a given block once.  The IPFS infrastructure distributes copies around a network for data redundancy and efficiency reasons, but this is unlike the situation with regular file systems in which multiple copies of the same content can exist under different names.  A second benefit is content permanence: writing new data to the system never changes the content associated with a given hash, because the hash is always unique.  Content stored in the system is thus immutable, which means it does not matter who serves it or how.  This affords the opportunity for efficient caching and preservation.  A third benefit is verifiability: any client can verify that a given piece of content is the expected content by recomputing the hash value for the data it gets back from a retrieval request.

## Network architecture

The network topology of IPFS is a flat, unstructured, fully decentralized network in which peers collectively provides a global storage and lookup service.  More specifically, the network implements the S/Kademlia distributed hash table (DHT) algorithm [@Baumgart2007-xr; @Maymounkov2002-lq], which defines how individual nodes find out about peer nodes as well as what to do when new nodes join the network and existing nodes disappear or fail.

At the peer level, the fundamental operations defined by a DHT are surprisingly few [@Zhang2013-eq]: essentially, there is only "ping" (to verify a node is still accessible), "store" (store a key-value pair), "find node" to return the node(s) that are closest to the requested key, and "find value", which is like "find node" but returns a value.  Key-value pairs in a DHT are stored in such a way that values are distributed around the network based on node identifiers; the identifiers are chosen from the same space as the keys, thus making individual nodes responsible for storing the values associated with particular keys.  In Kademlia, keys and node identifiers are 160-bit strings.  A distance metric is defined on identifier space (unrelated to physical network topology) such that proximity is assessed in the order of high bits to low bits.  When a new value is to be stored in the network, it is hashed to produce a 160-bit key, and a certain number of the leading bits of the key is used to derive the address of a node responsible for storing all values that begin with those same leading bits.

To enable peer-to-peer communications between heterogeneous devices with potentially different network underlays, the IPFS project developed a network communications stack that can seamlessly use multiple transports to provide an abstracted peer-to-peer connection API.  This is implemented in the form of a library called `libp2p`.  The library supports many transports including TCP, UDP, SCTP, UDT, WebRTC, SSH and other; multiple authentication protocols including TLS and SSH; self-describing wire protocols; NAT traversal; and encrypted channels. `libp2p` is agnostic with respect to transport protocols; it does not assume IP, and could be implemented on top of NDN.  In addition, `libp2p` can multiplex multiple communication operations over a single connection.  For example, it can multiplex multiple connections per peer, multiple network listening interfaces, multiple streams per protocol, and more.  Finally, once connected, connections can be reused from both ends.

IPFS can operate in partitioned networks.  A retrieval operation will work as long as any node in a network has the sought-after content.  Thanks to the uniqueness of hashes, rejoining partitioned networks naturally allows content previously stored separately to be available across the joined network, without fear of different content being stored under the same identifier while the networks were partitioned.  This has benefits for operation in areas with poor network connectivity and networks of mobile nodes.

## Data storage

Exchange of data between peers is accomplished using a custom protocol called BitSwap, based on the BitTorrent protocol.  The basic operation rests on a kind of "marketplace of data blocks" scheme, in which nodes have lists of data blocks they can offer and lists of blocks they want to acquire.  Unlike BitTorrent, the blocks in question are not limited to coming from a single source.  As discussed above, content stored on IPFS is uniquely identified and immutable, which means a peer does not care about the source of a given data block &ndash; every block is uniquely identified by its hash, so all a peer cares about is retrieving the block corresponding to a given hash.  In fact, the individual pieces of a large file (which will be decomposed into individual blocks) may be retrieved from different peers in the global IPFS network.  To make all of this work, the BitSwap protocol defines procesures for tracking block exchanges via distributed ledgers, as well as incentives for nodes to share data even when they do not seek to acquire anything in return.

The combination of DHT and BitSwap provides IPFS with a robust and efficient scheme for storing and distributing blocks of data across a large number of peers.  On top of this infrastructure, IPFS implements a _Merkle DAG_: a directed acyclic graph of objects where the links between objects are the hashes of the content stored in the nodes.  The following is a simple illustration of this concept:

<figure>
  <img width="500pt" src="merkle.svg">
  <!-- The extra <p> is needed to get centering to work on Safari. -->
  <figcaption><p align="center">Illustration of a DAG created between objects identified by hashes.</p></figcaption>
</figure>

The basic IPFS object structure is surprisingly simple:

```C++
          type IPFSLink struct {
  Name string;       // Name or alias of this link
  Hash Multihash;    // Cryptographic hash of target
  Size int;          // Total size of target
}

type IPFSObject struct {
  links []IPFSLink;  // Array of links
  data []byte;       // Opaque content data
}
```

Objects stored in IPFS Merkle DAGs are themselves data blocks with hashes, which is how IPFS stores structured objects (such as directories of files) and large files that have been decomposed into smaller blocks.  The format of the data stored in the `data` field is up to applications.

<!--
IPFS also defines data objects and a scheme to support versioning of objects.  The version object model is similar to Git's: there are blocks, lists, trees, and a _commit_ object that defines a snapshot in a version history.  The use of versioning is not automatic, but is a scheme available to applications.  It would be possible to build a version of Git that uses the IPFS object graph as its repository storage format.
-->

Merkle trees are used in other systems, including the Git revision control system, Dat, Bitcoin, and file systems such as ZFS.  The extension of the Merkle tree to a DAG used in IPFS is a generalization of that format.  It is a very flexible structure that allows many other types of data to be mapped directly to the IPFS Merkle DAG.  For example, Linked Data triples can be easily represented, as can cryptocurrency blockchains, key-value data stores, and others.

## Naming data in IPFS and IPNS

Unlike some other content-oriented storage schemes such as NDN, which allow users and applications to choose names in essentially any way they desire, in IPFS the "names" are unique fingerprints derived from the data itself via a cryptographic hashing function.  Anyone can compute the hash of a data block and ascertain whether the given hash matches the block &ndash; affording direct verification of data integrity.  However, the scheme as described so far is still incomplete:

* IPFS has values are long random strings of characters such as `XLF2ipQ4jD3UdeX5xp1KBgeHRhemUtaA8Vm`.  These are unsuitable for direct human use.

* Real-life scenarios do require the ability to associate different data with the same names, or (to put it another way) for a name to be reused.  For instance, when publishing a news website at a given address, it is important for an author to be able to show updated content without forcing users to access a different address every time.

* Without a way to define mutable data _somehow_, it would be impossible to communicate new data within the framework of IPFS itself: new content (or rather, the hashes for new content stored in the system) would have to be announced through some other means.  To understand why, consider something as simple as a text file containing a list of hashes for data.  Writing any changes to the file will lead to a new hash for the file itself, and thus a reader will _need to know the new hash_ in order to read the modified file &ndash; but how will the reader find out the new hash for the modified file?

This conundrum is solved by a combination of things: a scheme for self-certified names, and IPNS, the InterPlanetary Name Server, for mapping names to hashes.  The approach for self-certifying names is borrowed from another system called SFS (Self-certifying File System).  The scheme in IPFS for names uses public-key cryptography to sign the combination of a name and the hash to which the name refers, and publish this mapping via IPNS.  Clients can query IPNFS with a name and get back a result that has been signed with a digital signature, allowing a client to verify that the hash mapped to the name has not been altered.

## Data availability and persistence

The nodes in an IPFS network are the providers of storage.  However, it is not some kind of large cloud-based storage system.  When a node stores content in IPFS, it advertises to the network that it has content &ndash; but that content is on its local drive.  IPFS does not require every node to store all of the content ever published; in fact, it is an explicit design requirement that an IPFS client does not automatically download content that a user does not request.  Nodes are not required to "store other people's stuff", which is important for legal reasons.  Still, this raises a question: how _does_ content end up persisting?

The first element of the solution is the notion of _pinning_ content, which is how a client can tell IPFS it wants certain data to persist rather than allow it to be cleared at certain times (such as when a client reboots).  The second element is that nothing prevents users from creating nodes that keep their content online, nor in establishing collaborations with other nodes to host content on certain conditions.  Users can offer to host content, and in a large-enough network, content will end up being copied and replicated on multiple nodes.  There is also an ongoing effort to define `ipfs-cluster`, a tool to coordinate content distribution between IPFS nodes that will make this process easier.  Finally, a third major element of achieving persistence in IPFS is Filecoin, discussed below.

# Filecoin

Filecoin is a sister effort to IPFS aimed at providing incentives for users to provide storage.  Today, there exists a vast amount of unused disk space around the world in data centers and individual user's hard disks.  Filecoin is an electronic currency similar to Bitcoin that lets users offer a portion of their disk space and be paid for fulfilling storage requests on the Filecoin market.  Filecoin's proof-of-work function involves proof-of-retrievability, which requires nodes to prove they store a particular file.

The use of Filecoin incentivizes users to store data in IPFS.  If successful, it has the potential to discourage users from merely leeching the services provided by other users, and effectively solve a problem that has plagued many distributed peer-to-peer systems: leveraging humans' natural self-interest to encourage users to provide services for the good of the overall system.

# Additional reading

Being a relatively young project not based in an academic environment, there are unfortunately no publications describing IPFS in a comprehensive and up-to-date manner.  The following list of materials may help readers learn more.

1. The most detailed explanation of the entire architecture is Benet's 2014 white paper [@Benet2014-sp], but it is fairly technical and still unfinished in places.
2. A reasonably high-level introduction to IPFS can be found in a 2015 blog posting by Drake [@Drake2015-zg].
3. Another blog posting by Lundkvist and Lilic [@Lundkvist2016-ki] also provide useful high-level descriptions of IPFS.
4. After that, the best places to find out more are to read discussions on the IPFS forums at [https://discuss.ipfs.io/](https://discuss.ipfs.io/).



# References
