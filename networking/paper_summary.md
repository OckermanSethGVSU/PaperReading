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



## Jupiter Rising: A Decade of Clos Topologies and Centralized Control in Google’s Datacenter Network

Published: 2015

[Link](https://dl.acm.org/doi/pdf/10.1145/2829988.2787508)

**Main themes**

* Multi-stage Clos topologies built from commodity switch silicon can support cost-effective deployment for large clusters
* Central control mechanism is sufficient in most cases
* Modular hardware and simple software allows for scaling 

**Problem:** 10 years ago, cost and operational complexity associated with traditional datacenter network architectures was prohibitive; Traditional cluster architecture met scale needs but fell short in terms of overall performance and cost

**Summary**

```  This paper describes our experience with building five generations of custom data center network hardware and software leveraging commodity hardware components, while addressing the control and management requirements introduced by our approach``` 

General Structure

![image-20240906105508671](/home/seth/.var/app/io.typora.Typora/config/Typora/typora-user-images/image-20240906105508671.png)

** Fire House Version 1**

![image-20240906120543257](/home/seth/.var/app/io.typora.Typora/config/Typora/typora-user-images/image-20240906120543257.png)

*  8x10G switches for the fabric and 4x10G switches for ToRs

* Low radix of ToRs caused reliability issues

**Fire House Verison 1.1**

![image-20240906120400600](/home/seth/.var/app/io.typora.Typora/config/Typora/typora-user-images/image-20240906120400600.png)

* Version 1 + a custom datacenter cluster fabric (new scheme for the aggregation layer )
* Compact PCI chassis each with 6 independent linecards and a dedicated Single-Board Computer (SBC) to control the linecards using PCI
* Each ToR has 4 uplinks
* scale to 2x the number of machines while being much more robust to link failure
* Wiring was an issue

**Watch Tower**

* key idea was to leverage the next-generation merchant silicon switch chips, 16x10G, to build a traditional switch chassis with a backplane
* larger bandwidth density of the switching silicon also allowed us to build larger fabrics with more bandwidth to individual servers
* Cable bundling helped reduce fiber cost (capex + opex) by nearly 40%
* Still expensive

**Saturn** 

* principal goals were to increase bandwidth and maximum cluster scale
* Built using 24x10G merchant silicon building blocks
* could burst at 10Gbps across fabric

**Jupiter** 

![image-20240906121731711](/home/seth/.var/app/io.typora.Typora/config/Typora/typora-user-images/image-20240906121731711.png)



* 40GB burst bandwidth

**External Connectivity**

![image-20240906122525816](/home/seth/.var/app/io.typora.Typora/config/Typora/typora-user-images/image-20240906122525816.png)

### Software

**Neighbor Discovery** 

Used the configured view of cluster topology together with a switch’s local ID to determine the expected peer IDs of its local ports. It regularly exchanges its local port ID, expected peer port ID, discovered peer port ID, and link error signal. Doing so allows ND on both ends of a link to verify correct cabling.

**Firepath**

 Centralized topology state distribution, but distributed forwarding table computation

* Master constructs Link State Database which it then distributes to client
* Client use link stage database to calculate shortest path forwarding with ECMP and programs the hardware forwarding tables local to its switch
* redundant master instances on pre-selected spine switches