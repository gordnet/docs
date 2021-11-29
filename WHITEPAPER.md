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
- `visitor` - an entity that wishes to access some endpoint
- `bandwidth` - the data transfer capacity of a computer network in bits per second (Bps)
- `throughput` - the actual amount of data that is successfully sent/received over the communication link, measured in bits per second (Bps)
- `relay node` - a computer in the network that is relaying information between 2 parties


# 2. Bridge Request



[1]: https://apps.dtic.mil/sti/pdfs/ADA465464.pdf
[2]: https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.927.1044&rep=rep1&type=pdf
[3]: https://community.torproject.org/relay/types-of-relays/
