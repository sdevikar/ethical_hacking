# Notes

## IP addressing

- 32 BYTES of address. Commonly split in 4 tuples to a dotted notation for readability: X.Y.Z.W
- IP address logically consists of network part and host part
- There are 5 types of IP address classes. A, B, C, D and E
- A,B and C are unicast classes (i.e. they're used to address an individual hosts)
- Class D is Multicast class
- Class E is reserved for special IP addressing

## Class A IP Address

- Has 0 set as the most significant bit
- So the address range is 0.0.0.0 to 127.255.255.255
- The first tuple (e.g. 0 and 127 is network address) and the rest of it is host address
- If we represent this IP address as x.y.z.w, the x part is network and rest of it is host. This means, class A IP address is used for smaller networks with large number of hosts (flatter network)

## Class B IP Address

- Has 10 as the most significant bits
- So the address range is x80.0.0.0 (i.e. 128.0.0.0) - xbf.ff.ff.ff (128.255.255.255)
- First two tuples are network and last two tuples are host
- If we represent this IP address as x.y.z.w, the x and y part is network and rest of it is host. So, less flat

## Class C IP Address

- Has 110 as the most significant bits
- So the IP address range is 192.0.0.0 - 223.255.255.255
- If we represent this IP address as x.y.z.w, the x part is network and rest of it is host. This means, class C IP address is used for even wider networks with some hosts. i.e. even less flat

## Class D IP Address

- This is a multicast address
- IP addresses strart with the bits `1110` (i.e. most significant bits)
- When something is sent to a multicast address, all the hosts in the network
- range is 224.0.0.0 to 239.255.255.255  

## Ports

- Numbers associated with well known services running on a host e.g. ssh is associated with port 21
- These are documented in /etc/services on unix based hosts

## Ephemeral IP address

- A temporary sorts of IP address that client uses when it's talking to some other host (e.g. a server) on the network
- The idea is that, client doesn't have a service running that needs any port address
- But the port address is needed regardless for the server to communicate with the client
- After the communication is over, this port number is simply discarded

## TCP and UDP

- Protocols that work on top of IP prototocol
- IP guarantees data link layer functionality but it's not a "reliable" protocol. i.e. no retries and error checking
- This reliable part heavy lifting is done by TCP layer. Retries, error checking, etc. happen here
- UDP is just like IP, but with additional CRC checks

## IP Subnetting

We know that IP addresses are divided into network and host part. Subnetting has to do with dividing the host part further into sub networks.

### Natural masks

- As seen above network addresses have classes based on what part it the network address and what part is the hosts addresses
- Natural masks are the masks that keep the network part and zero out the host part
- For example, for class A network, first 8 bits are network part and the rest is host part. So, the natural mask will be 255.0.0.0 for class A network

### Subnet masks

- Allows for treating part of host address as a subnet
- For example, if a class A IP address was something like 10.5.0.20, we know that 10 is the network part and 5.0.20 is the host part. Natural subnet mask for this part is 255.0.0.0.
- But if we now use a "subnet mask" that's 255.255.0.0, it is the indicator that we're considering 10.5 as netowrk address and 0.20 as host address. Out of the 10.5 , 10 is network address and 5 is subnet address
- In other words, if we masked the IP address using this subnet mask, the result will be 10.5.0.0. That is, the host part is masked out

### Variable length subnet mask

- Host addresses are divided into subnets as above. And subnets can further be divided into sub-subnets and so on
- This division doesn't need to be symmetrical. i.e. not all subnets will be divided into sub-subnets, etc
- See class slides for the tree diagram

## Classless Internet Domain Routing

- There are no classes like A,B,C,D but rather the representation itself describes how IP address is divided into network and host part
- CIDR address is of format x.y.z.w / M , where M describes the number of bits that represent the network part. So, for example, 192.168.44.165/18 will mean that first 18 bits are network and the rest of the 14 are host 
- CIDR is used as a standard these days

## Routing

### Types of routing

- Next-hop routing: Routing table is based on next hop. i.e. we know what **host** we want to go to. And that's mapped against the router that will get you there. Routing table will contain this exact information
- Network specific: Routing table is based on destination **network** address. We know what **network** we want to go to (e.g. broadcast to each host in that network). The routing table will map the network against the router in this case.
- Host specific routing: Similar to next-hop routing, except the table will contain the exact IP address of host. This table will also be hybrind of next hop and network specific routing table.
- Default routing: Any destination that doesn't match anything else will be matched to this entry and the default router is mapped against it.

### Routing table fields

- Contains IP address of the host, subnet mask and next hop address, plus some flags and interface field (telling which outer link to follow)
- Anding the destination IP address with the subnet mask will give the network address and that's how packets are routed
- The idea is, whenever the packet arrives and needed to be routed:
  - the destination address is AND-ed with the subnet mask
  - If the result of the above matches the corresponding table entry, the packet is routed to the corresponding router or interface
  - If the result doesn't match, the anding happens with the next subnet mask entry in the table and steps are repeated
  - If the match doesn't happen even after processing all the entries in the table, the default routing path is used. For this entry, the subnet mask is 0.0.0.0. There's a corresponding destination 0.0.0.0 entry in the table and the default router address

### Netstat

- on linux `netstat -r` command will print the routing table

### Routing protocols

Routing protocols are all about keeping the routing tables updated.
There are two types of routing protocols, internal and external, depending on whether the communication is between routers within internal network (better known as Autonomous system)
For communication within AS, two protocols are used:

- Routing Information Protocol (RIP):
  - Uses distance vector routing using hop count
  - Distance vector routing protocol is slow because once a router updates its distance from something, it triggers a chain reaction
  - Also, counting to infinity is a serious drawback, if the link between two router dies.
  - Obsolete now.
  
- OSPF
  - Used widely for routing table updates within an AS
  - Routers use Dijkstra's algorithm for routing from point A to B
  - There are different types of packets, like Hello packet, to check if neighbor is up, Link state acknowledgement, (LSA), etc.

### Routing examples

- Let's say we have an IP address which we need to forward packets to. The routing table's job is to decide what network interface to forward packets to.
- The table looks something like this:
  
destination | subnet mask | interface
------------|-------------|----------
144.16.0.0 | 255.255.0.0 | eth0
144.16.64.0 | 255.255.224.0 | eth1
144.16.68.0 | 255.255.255.0 | eth2
144.16.68.64 | 255.255.255.224 eth3
default | 0.0.0.0 | eth1

- Our task here is to figure out what interface the IP address 144.16.68.131 should be routed to
- The way we do this is by anding the IP address with subnet mask and see if it matches the destination. If the destination matched the anded result, then we know the interface in the corresponding row should be used
  - Row 1: 144.16.68.131 & 255.255.0.0 => 144.16.0.0 == destination. So row 1 matches and eth0 can be used potentially. But let's keep going
  - Row2: 144.16.68.131 & 255.255.224.0 => 144.16.64.0 == destination. So, row 2 matches too and eth1 can be used 
  - Row3: 144.16.68.131 & 255.255.255.0 => 144.16.68.0 == destination. So, row 3 matches too and eth2 can be used 
  - Row4: 144.16.68.131 & 255.255.255.224 => 144.16.68.128 != destination. So, row 4 done not match and eth3 can NOT be used
  - default is used when no row matches
- In the above case, we have 3 potential candidates. But row#3 matches the most bits, so eth2 will be used. This is called the **longest prefix match** rule.

## Available tools

- nmap
  - Capable of scanning what hosts are on the network, what OS they're running, what ports are open and so on 
  - nmap can also be used to run vulnerability script using --script option
- nesus
    - TODO
- metasploit
  - open source penetration testing software (but paid) to exploit vulnerabilities
  - d