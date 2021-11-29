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


# 2. Decentralized Network Discovery

Each `node` in the network is connected (by default) to 10 `peers`. To boostrap the list, similar to the <cite>[Bitcoin P2P bootstrapping mechanism][4]</cite>, Bridgenet will first try to connect to peers that it has connected to before, saved in its local database. If no cached peers can be connected to, it will query a DNS seed to get a list of active peers. If none of those are reachable, it will fallback to a hardcoded peer address. Once a peer is reached, it will be queried to find additional peers.

Each node will send a heartbeat packet (of 10MB) to measure `throughput` periodically. Any peers that have a throughput slower than 1 standard deviation away from the average throughput will be dropped and new peers will be sought.


# 3. Bridge Request

When a `visitor node` wants to access the Bridgenet Network, it will first send a `bridge request` to each of its `peers`, these are the first layer connections. Each of them then sends a `bridge request` to each of their `peers` to build the second layer connections. Each of these sends a `bridge request` to each of their `peers` to get to the third and final layer connections. At this point, there's (potentially) 1,000 different paths, assuming that no 2 `nodes` are connected to the same `peers`. However, in practice, it's likely that there's significant overlap of `nodes`, which means the total number of unique paths will be less than the 1,000 theoretical maximum.

Each `bridge request` is a packet of 10MB, so that the `relay node` can measure the uplink `throughput`. A `bridge request response` is a 10MB packet that includes 

When a node receives a `bridge request`, it replies with 



[1]: https://apps.dtic.mil/sti/pdfs/ADA465464.pdf
[2]: https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.927.1044&rep=rep1&type=pdf
[3]: https://community.torproject.org/relay/types-of-relays/
[4]: https://bitcoin.stackexchange.com/a/32952/1151
