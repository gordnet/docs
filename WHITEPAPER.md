# Lily Whitepaper


# 1. Introduction

Lilynet is a decentralized trustless minimal-knowledge network that acts as a relay between two communicating parties via HTTP/HTTPS protocols. Specifically, when a user attempts to send a request to some website, the information can be intercepted and analyzed by various third parties (nefarious, Orwellian, etc.). Projects like <cite>[Tor][1]</cite> and <cite>[I2P][2]</cite> attempt to obfuscate the path between a user and his/her destination. In standard TCP/IP communication, both parties have information (specifically the IP address) of the other party. While Tor relay nodes remove this information, the problem is the lack of nodes due partially to the lack of incentive of being a relay. The figure below shows that there are currently 6,000 Tor relays and they are maintained in a <cite>[centralized list][3]</cite>, which makes it trivial for nefarious actors to block the services.

[<img src="https://user-images.githubusercontent.com/1019677/143939802-29eda65e-35a1-4d66-9c75-eea0a03fc409.png" width="50%"/>](https://user-images.githubusercontent.com/1019677/143939802-29eda65e-35a1-4d66-9c75-eea0a03fc409.png)

Lilynet aims to solve this problem by adhering to 3 principles:

1. The network should be completely decentralized
2. The market determines the price of privacy
3. The third


## 1.1 Definitions
To keep consistency in this proposal, the following definitions will be referred to:
- `visitor node` - the computer of an entity that wishes to access some endpoint
- `bandwidth` - the data transfer capacity of a computer network in bits per second (Bps)
- `throughput` - the actual amount of data that is successfully sent/received over the communication link, measured in bits per second (Bps)
- `relay node` - a computer in the network that is relaying information between 2 parties
- `peer` - a `relay node` that either a `visitor` or `relay node` is connected to
- `coin` - a unit of monetary value. Details TBD
- `exit node` - the `node` that will directly connect with the `destination`
- `destination` - the site or service attempting to be accessed anonymously


# 2. Decentralized Network Discovery

Each `node` in the network is connected (by default) to 10 `peers`. To boostrap the list, similar to the <cite>[Bitcoin P2P bootstrapping mechanism][4]</cite>, Bridgenet will first try to connect to peers that it has connected to before, saved in its local database. If no cached peers can be connected to, it will query a DNS seed to get a list of active peers. If none of those are reachable, it will fallback to a hardcoded peer address. Once a peer is reached, it will be queried to find additional peers.

Each node will send a heartbeat packet (of 10MB) to measure `throughput` periodically. Any peers that have a throughput slower than 1 standard deviation away from the average throughput will be dropped and new peers will be sought. Since a `peer` is a two-way connection, when the dropped `peer` attempts the heartbeat, it will find that it has been dropped and will find a new `peer`.


# 3. Bridge Request

When a `visitor node` wants to access the Bridgenet Network, it will first send a `bridge request` to each of its `peers`, these are the first layer connections. Each of them then sends a `bridge request` to each of their `peers` to build the second layer connections. Each of these sends a `bridge request` to each of their `peers` to get to the third and final layer connections. At this point, there's (potentially) 1,000 different paths, assuming that no 2 `nodes` are connected to the same `peers`. However, in practice, it's likely that there's significant overlap of `nodes`, which means the total number of unique paths will be less than the 1,000 theoretical maximum.

Each `bridge request` is a packet of 10MB, so that the `relay node` can measure the uplink `throughput`. When a node receives a `bridge request`, it replies with it's `throughput` and a price in `coin` per 100MB of data. The packet size of the response is 10MB so that the requesting `node` can ascertain if the reported `throughput` is valid (within some tolerance). The price mechanism will be covered in the [Price Discovery](#5.-Price-Discovery) section. Each `node` that sends a `bridge request` and receives a response calculates, based on some heuristic, which offer is the best (`throughput` / `coin`) and passes that backwards to any `node` that sent it a `bridge request`. Along with the pricing and `throughput` information, each node passes back it's public key.

![Cryptornet - Ref doc  - Bridge Request](https://user-images.githubusercontent.com/1019677/144675966-e98faf5f-b19a-4140-8be7-9d29d93d7dfe.png)


# 4. Proof-of-Relay

When the `originator` sends a packet to the first `node` in the `bridge`, it is first wrapped and encrypted 3 times. Along with sending the packet, the `originator` also broadcasts an `unlockKey` to the network via all of its peers. The `originator` sends a packet that has been encrypted with the public key of that node. When that packet is decrypted by the first `node`, it looks like:

```json
{
  "data": "[ENCRYPTED DATA BUFFER]",
  "unlockKey": "1234ABC"
}
```

The first `node`, then forwards along the value from `data` (`[ENCRYPTED DATA BUFFER]` in this case) to the second `node` in the `bridge`. It also broadcasts the `unlockKey` to it's peers, which continue to propagate that `unlockKey` to their peers, and so on. The packet that is sent to the second `node`, when decrypted by the second `node` looks similar:

```json
{
  "data": "[ANOTHER ENCRYPTED DATA BUFFER]",
  "unlockKey": "5678ABC"
}
```

The same actions apply and now the second `node` is revealing its `unlockKey` to its `peers`, and so on. The process goes on one more time with one difference. When the third `node` in the bridge decrypts the packet, it has:

```json
{
  "data": "[ORIGINAL DATA REQUEST]",
  "unlockKey": "90123ABC"
}
```

It forwards the `[ORIGINAL DATA REQUEST]` to the `destination` and reveals its `unlockKey` to all of its peers. Now there are 4 `unlockKeys` that the network is aware of. Each node races to combine these 4 signatures to unlock the reward (discussed in Section TBD).


# 5. Price Discovery

While not at the protocol level, each `node` will determine its own price per MB (in terms of `coin`). A `node` is incentivized to maximize its utilized `throughout`, so time will be a factor. Specifically, a node may start arbitrarily high (say 50 `coin` per MB), and decrease the price over some time period in an effort to entice other `nodes` to utilize it in a bridge.

Ideally, a `node` with higher `throughput` will be rewarded more highly than one with lower `throughput`. Given that `throughput` is relative, since network conditions are dynamic, each `node` will maintain a measure of its `throughput` to each of its `peers` by sending heartbeat packets and measuring response.

A client in the system that receives multiple `bridge responses`, each with their own price per MB, can choose whichever one meets the `Quality of Service` that it seeks. For low-latency applications, a client may choose to pay more, whereas for simple text based network content, a lower price may be more desirable.

# 6. Legal Implications of Relay Nodes
In general, being a relay for an anonymous network, such as TOR, is considered legal in the US. Complications arise when the relay is an exit node because all Tor users take on the IP addresses of their exit node and as a result, it is highly probable that an exit node will be used at some point for illegal purposes.
For exit node operators in the US, the Tor network allows the operators to adopt a reduced exit policy which is simply a configuration of the exit node "whereby it will only deliver data to specified ports in the Internet, and the ports are selected as those that will not deliver to sites that allow illegal file sharing." For those that are not using a reduced exit policy, it is vital to be aware of the legal implications surrounding exit nodes.
For example, The Protection of Children Against Sexual Exploitation Act prohibits the interstate transportation, distribution and receipt of visual images of child pornography “by any means including by computer.” However, the Supreme Court, in United States v. X-Citement Video, Inc., held that distribution entities must have specific knowledge of the pornographic images for the entity to be held liable under this statute. Since an exit node passes decrypted data, it is hard to prove that an exit node knowingly distributed such content and hence, according to Electronic Frontier Foundation(EFF), should not be held liable. 
In response to the copyright issues that arose as a result of the emerging popularity of file-sharing on the Internet, Congress enacted the Digital Millennium Copyright Act (DMCA) in 1998. Because Tor can be used to facilitate the transfer of copyrighted files without detection, exit Nodes can be theoretically subjected to DMCA laws. The most likely theory to apply DMCA liability to Tor node operators is the theory of contributory infringement, which states that a developer “knowingly” and “materially” facilitated the distribution of the copyright material. The best way for a Tor exit node to safeguard themselves against DMCA liabilities is to apply provisions set out in 17 U.S.C. §512(a). This provision states that an ISP can be held liable for copyright infringment if the ISP: (1) possesses the right and ability to supervise the infringing conduct and (2) has an obvious and direct financial interest in the exploitation of the copyrighted materials. Tor exit nodes, on the other hand, gain no financial incentives with the "explotation of copyrighted material" but as a matter of fact, incur costs in the form of reduced internet speed and bandwith and processing time when running a relay node. In essense, Tor networks are likely entitiled to safe harbor under the DMCA. 
Finally, the First Amendmenet is also vital in safeguarding exit nodes against litigations and prosecutions from the US government. The Supreme Court has been consistently protecting anonymous free speech. The legality of the anonymity networks centers on whether or not there must always be a way to identify a speaker. "Just as the First Amendment protects a right to speak, so also, it protects the right to be silent and refrain from speaking." "Speaking" can most definitely be extended to the act of transferring data and information. While individuals have a First Amendment right to anonymous speech on the Internet, that right is subject to limitation. There are two substantial limitations to the First Amendment’s protections. First, the First Amendment only protects against invasive or coercive governmental activities. This means that if a relay node who was served a DMCA notice voluntarily relinquishes information concerning a users’ identity, there are few protections available. Second,  anonymous speech remains subject to certain limitations such as "fighting words, incitement, defamation, and obscene speech are all unprotected categories of speech." Furthermore, not all speech equally implicates First Amendment protections. For instance, when Internet users share files, the First Amendment interest implicated is minimal, since file-sharers’ ultimate aim is not to communicate a thought or convey an idea, but rather to obtain copyrighted music or movies for without cost. As a result, so long as the anonymous network is used for expressive activity, it is protected under the First Ammendment.




[1]: https://apps.dtic.mil/sti/pdfs/ADA465464.pdf
[2]: https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.927.1044&rep=rep1&type=pdf
[3]: https://community.torproject.org/relay/types-of-relays/
[4]: https://bitcoin.stackexchange.com/a/32952/1151
