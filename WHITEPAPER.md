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

Each `bridge request` is a packet of 10MB, so that the `relay node` can measure the uplink `throughput`. When a node receives a `bridge request`, it replies with it's `throughput` and a price in `coin` per 100MB of data. The packet size of the response is 10MB so that the requesting `node` can ascertain if the reported `throughput` is valid (within some tolerance). The price mechanism will be covered in the [Price Discovery](#4.-Price-Discovery) section. Each `node` that sends a `bridge request` and receives a response calculates, based on some heuristic, which offer is the best (`throughput` / `coin`) and passes that backwards to any `node` that sent it a `bridge request`. Consider the example below.

%3CmxGraphModel%3E%3Croot%3E%3CmxCell%20id%3D%220%22%2F%3E%3CmxCell%20id%3D%221%22%20parent%3D%220%22%2F%3E%3CmxCell%20id%3D%222%22%20value%3D%22Satoshi%20(Originator)%22%20style%3D%22ellipse%3BwhiteSpace%3Dwrap%3Bhtml%3D1%3B%22%20vertex%3D%221%22%20parent%3D%221%22%3E%3CmxGeometry%20x%3D%2240%22%20y%3D%22160%22%20width%3D%22120%22%20height%3D%2280%22%20as%3D%22geometry%22%2F%3E%3C%2FmxCell%3E%3CmxCell%20id%3D%223%22%20value%3D%22Node%201%22%20style%3D%22rounded%3D1%3BwhiteSpace%3Dwrap%3Bhtml%3D1%3B%22%20vertex%3D%221%22%20parent%3D%221%22%3E%3CmxGeometry%20x%3D%22200%22%20y%3D%2280%22%20width%3D%22120%22%20height%3D%2240%22%20as%3D%22geometry%22%2F%3E%3C%2FmxCell%3E%3CmxCell%20id%3D%224%22%20value%3D%22Node%202%22%20style%3D%22rounded%3D1%3BwhiteSpace%3Dwrap%3Bhtml%3D1%3B%22%20vertex%3D%221%22%20parent%3D%221%22%3E%3CmxGeometry%20x%3D%22200%22%20y%3D%22160%22%20width%3D%22120%22%20height%3D%2240%22%20as%3D%22geometry%22%2F%3E%3C%2FmxCell%3E%3CmxCell%20id%3D%225%22%20value%3D%22Node%2010%22%20style%3D%22rounded%3D1%3BwhiteSpace%3Dwrap%3Bhtml%3D1%3B%22%20vertex%3D%221%22%20parent%3D%221%22%3E%3CmxGeometry%20x%3D%22200%22%20y%3D%22320%22%20width%3D%22120%22%20height%3D%2240%22%20as%3D%22geometry%22%2F%3E%3C%2FmxCell%3E%3CmxCell%20id%3D%226%22%20value%3D%22%22%20style%3D%22endArrow%3Dclassic%3Bhtml%3D1%3Brounded%3D0%3BentryX%3D0%3BentryY%3D0.5%3BentryDx%3D0%3BentryDy%3D0%3B%22%20edge%3D%221%22%20source%3D%222%22%20target%3D%223%22%20parent%3D%221%22%3E%3CmxGeometry%20width%3D%2250%22%20height%3D%2250%22%20relative%3D%221%22%20as%3D%22geometry%22%3E%3CmxPoint%20x%3D%22410%22%20y%3D%22380%22%20as%3D%22sourcePoint%22%2F%3E%3CmxPoint%20x%3D%22460%22%20y%3D%22330%22%20as%3D%22targetPoint%22%2F%3E%3C%2FmxGeometry%3E%3C%2FmxCell%3E%3CmxCell%20id%3D%227%22%20value%3D%22%22%20style%3D%22shape%3Dimage%3Bhtml%3D1%3BverticalAlign%3Dtop%3BverticalLabelPosition%3Dbottom%3BlabelBackgroundColor%3D%23ffffff%3BimageAspect%3D0%3Baspect%3Dfixed%3Bimage%3Dhttps%3A%2F%2Fcdn2.iconfinder.com%2Fdata%2Ficons%2Fessential-web-5%2F50%2Fmore-dot-tripple-many-detail-128.png%3Bdirection%3Dsouth%3BfontSize%3D12%3B%22%20vertex%3D%221%22%20parent%3D%221%22%3E%3CmxGeometry%20x%3D%22235%22%20y%3D%22230%22%20width%3D%2250%22%20height%3D%2250%22%20as%3D%22geometry%22%2F%3E%3C%2FmxCell%3E%3C%2Froot%3E%3C%2FmxGraphModel%3E



# 4. Price Discovery



[1]: https://apps.dtic.mil/sti/pdfs/ADA465464.pdf
[2]: https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.927.1044&rep=rep1&type=pdf
[3]: https://community.torproject.org/relay/types-of-relays/
[4]: https://bitcoin.stackexchange.com/a/32952/1151
