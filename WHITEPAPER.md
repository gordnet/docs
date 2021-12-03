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

# 6. 



[1]: https://apps.dtic.mil/sti/pdfs/ADA465464.pdf
[2]: https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.927.1044&rep=rep1&type=pdf
[3]: https://community.torproject.org/relay/types-of-relays/
[4]: https://bitcoin.stackexchange.com/a/32952/1151
