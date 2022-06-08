# Lab01 Basic Networking

Table of Contents:
- [Lab01 Basic Networking](#lab01-basic-networking)
  - [Introduction to IP Addresses v4](#introduction-to-ip-addresses-v4)
    - [Important concepts](#important-concepts)
    - [Task-A](#task-a)
      - [Prerequisites](#prerequisites)
      - [Tasks](#tasks)
    - [Questions](#questions)
  - [IPv4 Routing](#ipv4-routing)
    - [Important Concepts](#important-concepts-1)
    - [Task-B](#task-b)
      - [Prerequisites](#prerequisites-1)
      - [Tasks](#tasks-1)
      - [Questions](#questions-1)

## Introduction to IP Addresses v4 

### Important concepts
- IP address is an identifier of a network interface of a node.
  
  For example, if you own a laptop (Windows OS) that has multiple networking cards, e.g., an Ethernet interface and a Wi-Fi interface, then you can have two physical IP addresses as follows.

  ``` 
  PS C:\Users\AICOMS\Documents\ardimas\PROJECTS\cpen6234001> ipconfig
  
  Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::b4af:fe3:3a56:4f85%12
   IPv4 Address. . . . . . . . . . . : 10.25.130.200
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.25.130.1

  Wireless LAN adapter Wi-Fi:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::5d56:9156:d0ab:f68c%19
   IPv4 Address. . . . . . . . . . . : 10.25.155.111
   Subnet Mask . . . . . . . . . . . : 255.255.254.0
   Default Gateway . . . . . . . . . : 10.25.154.1

  ```

  An IPv4 address is a 32-bit number that is usually written as four 8-bit decimal numbers seperated by dots. 
  Each decimal number is referred to as an octet.
  Each octet ranges from 0 to 255.

- An IPv4 address consists of two parts, which are a network address and a host address.
    
  To obtain the network address please see the illustration below.
  
  ```
   IPv4       10   .   25   .  130   .  200       --> decimal notation
           00001010.00011001.10000010.11001000    --> binary notation

  subnet     255   .  255   .  255   .   0        --> decimal notation
   mask    11111111.11111111.11111111.00000000    --> binary notation


  -------------------------------------- XOR

  network  00001010.00011001.10000010.00000000
  address

  ```

  To obtain the host address please see the illustration below. 

  ```
   IPv4       10   .   25   .  130   .  200       --> decimal notation
           00001010.00011001.10000010.11001000    --> binary notation

  subnet     255   .  255   .  255   .   0        --> decimal notation
   mask    00000000.00000000.00000000.11111111    --> negated binary
                                                      notation

        -------------------------------------- XOR

   host    00000000.00000000.00000000.11001000
  address         
  ```

  An important takeaway from the illustrations above is that by knowing the network address we can determine which IPv4 addresses that can directly communicate to each other by only using a layer-2 device, such as switch.
  <strong>In our example above, it means that we can connect hosts having the IPv4 addresses from 10.25.130.0 to 10.25.130.255 over switches, and they are reachable from each other. </strong>

- The subnet mask is usually written by a CIDR notation.
  
  For example, the subnet mask 255.255.255.0 can be written as /24 coming from the fact that it denotes the number of bit 1s in the subnet mask. 
  Therefore, a network interface with the IPv4 address of 10.25.130.200 and the subnet mask of 255.255.255.0 can be written as 10.25.130.200/24.

- There are public IPv4 addresses and private IPv4 addresses.
  
  There are two types of IPv4 addresses.
  That is, one type of IPv4 addresses that can only be used in a local area network (LAN), e.g., home network or internal office network, which is referred to as <strong>private IP addresses</strong>.
  Examples of private IPv4 addresses are:
  - 10.0.0.0/8,
  - 172.168.0.0/12, and
  - 192.168.0.0/16.
  
  Note that these IPs are not routable in the public Internet.
  The other IPv4 addresses are referred to as <strong>public IP addresses</strong>.

- Reserved IPv4 addresses
  
  In the LAN, there are two important reserved IP addresses which are network address and broadcast address.
  The network address is the first address in the subnet. 
  And, the broadcast address is the last address in the subnet, which as the name implies, it is an IPv4 address that is used to broadcast message to all nodes in the same subnet.
  For example, with regards to our previous example, the network address and the broadcast address are 10.25.130.0 and 10.25.130.255, respectively.
  Other important reserverd IP addresses can be found in the [Wikipedia](https://en.wikipedia.org/wiki/Reserved_IP_addresses).

### Task-A

#### Prerequisites
- Setting an IPv4 address in Linux
  
  There are two common ways to set an IPv4 address to a network interface in Linux. 
  ```sh
  # ifconfig <interface> <ipv4> netmask <subnet mask>
  ifconfig eth0 192.128.122.2 netmask 255.255.255.0
  ```

  ```sh
  # ip address add <ipv4/cidr> dev <interface>
  ip address add 192.168.122.2/24 dev eth0
  ```
  
- Most L2-switches do not require any configuration to enable reachability of hosts in the same subnet in a LAN.

#### Tasks
1. Build the following topology.
   
   ![First Topology](./figs/screenshot-1654605552343.svg)

2. Build the following topology
   ![Second Topology](./figs/screenshot-1654608910546.svg)

   
### Questions
1. From Task-A-1, can they ping each other? Why can they ping each other? Show me at least a proof to support your answer to the why question!
2. From Task-A-2, can they ping each other? Why can they ping each other? Show me at least a proof to support your answer to the why question! Also, what is wrong with the topology?
3. Extend the topology from Task-A with an additional L2 switch such that you have three subnets such that the first subnet is approximately twice the size of the other two subnets. (Note that a subnet size denotes the maximum number of hosts in the subnet.)


## IPv4 Routing 

### Important Concepts
- Lookup table and the longest prefix match
  
  IPv4 addressing uses a lookup table and the longest prefix match rule to direct an incoming packet to the correct interface.
  For example, as shown in the table below, it is clear that a traffic having destination IP range of 192.168.10.0/24 (e.g., 192.168.10.2) will be directed to the interface F0.
  Meanwhile, due to the longest prefix match rule, any traffic having the destination IP range of 10.1.1.0/25 will be directed to the interface F2 instead of F1.
  For example, a traffic with the destination IP address of 10.1.1.64 will be forwarded to the interface F2.
  In addition, a look up table will usually contain a default route, for example 0.0.0.0/0 denotes that if a traffic's destination IP address is not specified in the table (e.g., 172.168.10.2), then the router will forward it to the interface F3.

  ```
    | Destination IP range | Interface |
    |----------------------|-----------|
    | 192.168.10.0/24      | F0        |
    | 10.1.1.0/24          | F1        |
    | 10.1.1.0/25          | F2        |
    | 0.0.0.0/0            | F3        |
  ```

- Network Address Translation (NAT)
  
  Recall that the private IP addresses are not routable in the public Internet. 
  Therefore, in order for a host having a private IP address to reach the public Internet, then the private IP address is translated to a public IP address.
  In practice, the private IP address is replaced by the router's interface that faces public Internet and has a public IP address.
  For example as shown in the figure below, the local computer's source IP address (172.16.10.3) will be replaced with the router's public IP address (182.253.160.140).
  

  ```
  +----------+             +------+             +--------+
  |   Local  |             |Router|             | Public |
  | Computer +------------>|      +------------>|Internet|
  +----------+             +------+             +--------+
              172.16.10.3       182.253.160.140
  ```

### Task-B

#### Prerequisites

- Setting IPv4 address of a Cisco router's interface
  
  1. Go to the terminal configuration menu
     ```
     R1#configure terminal
     ```
  2. Choose the interface to be configured
     ```
     R1(config)#interface FastEthernet0/0
     ```
  3. Add IP address 
     ```
     R1(config-if)#ip add 10.1.1.1 255.255.255.0
     R1(config-if)#no shut
     ```
  4. Validate
     ```
     R1(config-if)#end
     R1#show ip interface brief
     ```

- Setting routing table of a Cisco router
  
  ```
  R1#configure terminal
  R1(config)#ip route <destination ip address> <subnet mask> <default gateway interface>
  ```

  Note that in order to set a default routing, use 0.0.0.0 for both the destination ip address and subnet mask.

- Setting source NAT in a Cisco router
  ```
  R1#configure terminal
  R1(config)#interface <interface in the nat inside>
  R1(config-if)# ip nat inside
  R1(config)#interface <interface in the nat outside>
  R1(config-if)# ip nat outside
  R1(config-if)# exit
  R1(config)#access-list 100 permit ip any any
  R1(config)#ip nat inside source list 100 int <interface in the nat outside> overload
  ```
  Note that the interface in the nat inside is the one in the LAN.

  

- Setting default route in Linux
  ```sh
  route add default gw <IP-ADDRESS> <INTERFACE-NAME>
  ```
  or,
  ```
  ip route add default via <gateway IP address> dev <interface name>
  ```

#### Tasks
1. Build the following topology.
   ![example](./figs/screenshot-1654620542689.svg)

   where the NAT1 IP address is 192.168.122.1 (only true for those who use the remote server).
   It might be true for those who use GNS3 installed in your machine. 
   Look for the virbr0 interface in your terminal, and see the IP address.
   For the router R1, you could use the Cisco router 3745 whose image I have already gave you before.
2. From Task-A-2, please replace the switch at the top with a router and connect+add a NAT from the router. Then, configure the router properly such that all hosts can ping each other and all hosts can access the Internet. 
3. (Challenge) From Task-B-1, please replace the router with a Linux machine and enable Internet connection from all hosts

#### Questions
1. Why can all computers in the Task-A-2 topology now ping each other?
2. Suppose that you can perform a source NAT with a Linux machine, how would you practically put it into a use?
