# Lab02 Linux Network Namespace and Virtual Switch

Table of Contents:
- [Lab02 Linux Network Namespace and Virtual Switch](#lab02-linux-network-namespace-and-virtual-switch)
  - [Requirements](#requirements)
  - [Linux Network Namespace](#linux-network-namespace)
    - [Introduction](#introduction)
    - [Important Commands](#important-commands)
    - [Questions](#questions)
  - [Virtual Ethernet](#virtual-ethernet)
    - [Introduction](#introduction-1)
    - [Important Commands](#important-commands-1)
    - [Questions](#questions-1)
  - [Virtual Switch](#virtual-switch)
    - [Important Commands](#important-commands-2)
    - [Questions](#questions-2)
  - [Docker](#docker)
    - [Important Commands](#important-commands-3)
    - [Questions](#questions-3)
  - [Docker Compose](#docker-compose)
    - [Important Commands](#important-commands-4)
    - [Questions](#questions-4)
  - [Bonus: Load Balancer](#bonus-load-balancer)

## Requirements

I'm running the following commands with:
- 5.13.0-1022-aws Linux kernel,
- Ubuntu 20.04.4 LTS OS,
- 5.5.0-1ubuntu1 iproute2 command,
- Docker version 20.10.17, and
- Docker Compose version v2.6.0

## Linux Network Namespace

### Introduction 
Based on [Namespace Manual](https://man7.org/linux/man-pages/man7/namespaces.7.html), a namespace wraps a global system resource in an abstraction that makes it appear to the processes within the namespace that they have their own isolated instance of the global resource.
There are several types of Linux namespaces, i.e.,:
- cgroup,
- ipc,
- <strong>network</strong>,
- mount,
- pid,
- time,
- user, and
- uts.

In this lab, we will focus on the Linux network namespace (NS) which tightly relates to our discussion so far.
The network namespace virtualizes network-related resources, e.g., network stack, routing table, firewall, etc.
If it helps, I have the following visualization whenever I think about a network namespace. 
Imagine that you'll have an isolated environment (NS<em>i</em>) where each environment <em>i</em> has their own firewall configuration, arp table, and routing table.
In addition, you can attach multiple virtual interfaces (denoted by question marks '?').


```
+------------------------------+
|                              |
|      NSi                     |
|     +-----------------+      |
|     |                 |      |
|     |  +--------+     |      |
|     |  |Firewall|   +-+      |
|     |  +--------+   |?|      |
|     |               +-+      |
|     |  +-------+      |      |
|     |  |  ARP  |    +-+      |
|     |  | Table |    |?|      |
|     |  +-------+    +-+      |
|     |                 |      |
|     |  +-------+      |      |
|     |  |Routing|      |      |
|     |  | Table |      |      |
|     |  +-------+      |      |
|     |                 |      |
|     +-----------------+      |
|                              |
| Host Computer                |
+------------------------------+

```

### Important Commands

Please run the following commands as a root.

- Create a Linux network namespace
    
    ```sh
    # ip netns add <ns name>
    ip netns add red
    ```
- Enter into the network namespace
  
    ```sh
    # ip netns exec <ns name> bash
    ip netns exec red bash
    ```

- Identify if you are inside the namespace
  
    ```sh
    ip netns identify
    ```

- To exit the namespace

    ```sh
    exit
    ```
- To list active network namespaces
  
    ```sh
    ip netns ls
    ```

- To delete a network namespace
    
    ```sh
    # ip netns del <ns name>
    ip netns del red
    ```

### Questions
1. Use the command `ip route show` to compare the routing table in a network namespace and that in a host machine
2. Use the command `ip neigh` to compare the routing table in a network namespace and that in a host machine
3. Use the command `iptables -L` to compare the firewall rule in a network namespace and that in a host machine
4. Please create a dummy file in a host machine and see if you can see the same file inside a network namespace.

## Virtual Ethernet
### Introduction
Now, suppose that you want to connect multiple network namespaces.
You need a virtual cable as the representation of a physical cable to connect two networking devices.
In Linux, the basic virtual interface is called `veth` ([Please refer to here](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking#veth)).
Note that `veth` must be created in pairs.
For example, the pairs `veth-r` and `veth-g` are created to connect the `red` and the `green` namespace.


```
+------------+                    +-------------+
|            |veth-r        veth-g|             |
|          +-+-+                +-+-+           |
|    red   |   +----------------+   |  green    |
|          +-+-+                +-+-+           |
|            |                    |             |
+------------+                    +-------------+
```

The intuition is similar to a physical interface.
That is, if one interface is down, e.g., `veth-r` is down, then the communication cannot be established.
Therefore, you need to make sure that each interface is:
- up,
- assigned by an IP address in the same subnet, and
- attached to a network namespace.

### Important Commands

- To create a pair `veth`
  
    ```sh
    # ip link add <interface name 1> type veth peer name <interface name 2>
    ip link add veth-r type veth peer name veth-g
    ```

    Type `ip link show` to see both interfaces in the list

- To attach a `veth`
  
    ```sh
    # ip link set <interface name> netns <netns name>
    ip link set veth-r netns red
    ```

    Type `ip link show` and compare it with the previous result

- To activate a `veth`
  
    ```sh
    # ip link set dev <interface name> up
    ip link set dev veth-r up
    ```
    Note: you know where to run this command, right? 

- To assign an IP address
  
    ```sh
    # refer to the previous lab
    ip address add 192.168.1.1/24 dev veth-r
    ```

### Questions
1. Try to enable a connection between two network namespaces.
2. (Challenge) Try to create a chain of 3 network namespaces and enable connections between them.


## Virtual Switch

A virtualization does not stop at the the virtual ethernet, but we also have a Linux virtual switch.
Let's start with a L2 switch. 
We'll use the [Linux bridge](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking#bridge).
The easiest way to understand the Linux bridge is that it is another network namespace where we can attach an interface, and the Linux bridge will forward incoming traffic.
However, we can't access its shell.

### Important Commands

- To create a virtual bridge
  
    ```sh
    # ip link add <virtual bridge name> type bridge
    ip link add v-bridge type bridge
    ```

- To attach a `veth` to the virtual bridge
  
    ```sh
    # ip link set <veth name> master <virtual bridge name>
    ip link set veth-br master v-bridge
    ```

### Questions
1. Create a topology where it consists of a virtual switch that connects two network namespaces located in the same subnet.
2. Enable the Internet connection from the network namespaces above.

## Docker

### Important Commands
- To pull a Ubuntu image
  
    ```sh
    docker pull ubuntu
    ```

- To run a persistent ubuntu container
  
    ```sh
    # docker run -it -d --name <container name> ubuntu /bin/bash
    docker run -it -d --name ubuntu1 ubuntu /bin/bash
    ```

- To list a running container
  
    ```sh
    docker container ls
    ```

- To access a running container's shell
  
    ```sh
    # docker exec -it <container name> bash
    docker exec -it ubuntu1 bash
    ```

### Questions
1. Run multiple docker containers from the ubuntu image. Can they ping each other? (Please install the command `ping` if the command does not exist yet.) How does the topology look like?
2. Can you ping the other container using its container name? Why?
3. Try to create a file in the host computer and in all docker containers. Can they see each other file? What is the implication of it? What technology that enables it?

## Docker Compose

### Important Commands
- To run multiple services:
    
    1. Create a file named `docker-compose.yml`

        ```
        services:
            ubuntu1:
                image: ubuntu
                container_name: ubuntu1
                command: ["sleep","infinity"]
            ubuntu2:
                image: ubuntu
                container_name: ubuntu2
                command: ["sleep","infinity"]
        ```

    2. Run
   
        ```sh
        docker compose up -d
        ```

    3. Validate

        ```sh
        docker compose ps
        ```

- To stop

    First, make sure that you are in the same directory as the `docker-compose.yml` file.
    ```sh
    docker compose stop
    ```

### Questions
1. Can they ping each other by using the container name? How? And, what is the implication?

## Bonus: Load Balancer

To be added later