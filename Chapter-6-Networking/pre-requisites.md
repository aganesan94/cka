# Linux Networking Basics

## Switching

* Let us say there are 2 computers, Computer A and Computer B
* Computer A needs to talk to Computer B
* Computer <-------> Network Switch via ethernet port <-----------> Computer B
* Executing the following command provides the interface information for each host.

```
$ ip link
```
* Let us assume the following IP Addresses for the Network-1.

  * Network Switch - 192.168.1.0
  * Computer A - eth0 - 192.168.1.10
  * Computer B - eth0 - 192.168.1.11


```
ip addr add 192.168.1.10/24 dev eth0
ip addr add 192.168.1.11/24 dev eth0
```


* Let us assume the following IP Addresses for the Network-2.

  * Network Switch - 192.168.2.0
  * Computer C - eth0 - 192.168.2.10
  * Computer D - eth0 - 192.168.2.11

```
ip addr add 192.168.2.10/24 dev eth0
ip addr add 192.168.2.11/24 dev eth0
```


## How to make sure Network-1 can connect with Network-2?

* This can be done thru a router. 
* There can be many routers which the networks are connected to.
* Hence there is a need to have a gateway, a door to the outside network.

## How can the systems connect via a gateway?

* Running the route command displays the current routers which are configured

```
$ route
```
### Have Computer B send and receive packets to Computer C
\
SSH into Computer B add the following route 

```
ip route add 192.168.2.0/24 via 192.168.1.1
```

### Have Computer C send and receive packets to Computer B

```
ip route add 192.168.1.0/24 via 192.168.2.1
```

### Have Computer C send and receive packets thru the internet which has an IP address of 172.217.194.0

```
ip route add 172.217.194.0/24 via 192.168.2.1
```

### How can Netbwork-2 connect with any other network?

If the same router is OK to be used just add a default gateway

```
ip route add default via 192.168.2.1
```

## Setting up a linux host as a router?

#### Computer A
IP: 192.168.1.5
Network IP: 192.168.1.0

#### Computer B
IP: 192.168.1.6
Network IP: 192.168.1.0

IP: 192.168.2.6
Network IP: 192.168.2.0

#### Computer C
IP: 192.168.2.5
Network IP: 192.168.2.0

### If Computer A needed to talk to Computer C

#### On Computer A

```
ip route add 192.168.2.0/24 via 192.168.1.6
```

#### On Computer C

```
ip route add 192.168.1.0/24 via 192.168.2.6
```