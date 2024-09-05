## A Scalable Commodity Data Center Network Architecture

Published: 2008

[Link](http://ccr.sigcomm.org/online/files/p63-alfares.pdf)  

**Main Idea**: how to leverage commodity Ethernet switches to support massive clusters 

**Paper goal**: Using commodity hardware, achieve: scalable interconnect bandwidth, econ of scale, backwards comparability

**Achievements:**

* Using fat-tree architecture, achieve full bisection bandwidth in 10k cluster nodes
* Lower cost than existing solutions with more bandwidth

### Intro 

*inter  = between | intra = within* 

* The main bottleneck in clusters is often inter-node communication bandwidth
* Two main choices for communication fabric
  * Specialized hardware (e.g. InfiniBand) - not compatible with TCP/IP applications
  * Commodity hardware - however aggregative bandwidth scales poorly and achieving high bandwidth incurs non linear cost

* With commodity hardware
  * often oversubscribed (two nodes connected to same physical switch can use full bandwidth, else - not so much)

### Background 

*Uplink: to router / Internet*  | *Port: to Ethernet / local-subnet*

* Networking architectures often consist of two or three level trees of switches and routers 

  ![image-20240904114647226](/home/seth/.var/app/io.typora.Typora/config/Typora/typora-user-images/image-20240904114647226.png)

```
oversubscription: the ratio of the worst-case achievable aggregate bandwidth among the end hosts to the total bisection bandwidth of a particular communication topology. An oversubscription of 1:1 indicates that all hosts may potentially communicate with arbitrary other hosts at the full bandwidth of their network interface (e.g., 1 Gb/s for commodity Ethernet designs). An oversubscription value of 5:1 means that only 20% of available host bandwidth is available for some communication patterns.
```

**Equal Cost Multi-Path Routing**

* uses static load balancing
* multiplicity limited to 8-16

**Fat Tree**

```
We adopt a special instance of a Clos topology called a fattree [23] to interconnect commodity Ethernet switches. We organize a k-ary fat-tree as shown in Figure 3. There are k pods, each containing two layers of k/2 switches. Each k-port switch in the lower layer is directly connected to k/2 hosts. Each of the remaining k/2 ports is connected to k/2 of the k ports in the aggregation layer of the hierarchy.
```

![image-20240904115502201](/home/seth/.var/app/io.typora.Typora/config/Typora/typora-user-images/image-20240904115502201.png)

**Advantages** 

* all switching elements identical - cheap 
* rearrangeable non-blocking, meaning that for arbitrary communication patterns, there is some set of paths that will saturate all the bandwidth available to the end hosts in the topology.

**Issues**

* IP/Ethernet typically build a single routing path between each source and destination, creating bottlenecks
* Wiring complexity

### Architecture 

*IP 8.8.8.8/24 -> prefix 8.8.8., suffix 8.*

Introduce two level routing tables that  that spread outgoing traffic based on the low-order bits of the destination IP address

TCAM stores routing tables that point to addresses in RAM with next-hop / output ports 

![image-20240904151543322](/home/seth/.var/app/io.typora.Typora/config/Typora/typora-user-images/image-20240904151543322.png)

Routing scheme that uses two level tables, providing multiple paths for any given packet. Core still only have one option for each pod, but aggregation has many 

**Implements Flow Classification with Dynamic port reassignment by**

1. recognizing packets from same flow and use same port
2. Periodically reassign a minimal number of flow output ports to minimize any disparity between the aggregate flow capacity of different ports.

Central Scheduler assigns uncontested path for flows tha exceed a threshold in size

Has component failure detection and a degree of redundancy to tolerate it 

![image-20240904153951681](/home/seth/.var/app/io.typora.Typora/config/Typora/typora-user-images/image-20240904153951681.png)