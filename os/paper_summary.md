## The Design and Implementation of a Log-Structured File System

Published: 1991

**Main idea:** create a log-based file structure that writes data to disk sequentially. The main memory is used as a buffer for write operations and then the buffer is written to disk in a single disk operations.  

**Achievements** 

* Improves small file writes
* Matches long read/ write
* 70% bandwidth vs typical 5-10%

### Introduction 

* CPU speed increasing while disk access times are not (only a little)
* Based on the assumptions that most reads will be satisfied by main memory, therefore disk traffic is dominated by writes

Log file systems (LFS) write to disk in sequential structure called logs, eliminating most seek time and improving recovery time 

Main challenge with LFS - ensuring lots of free space for new data 

Solution: Split data into segments and use segment cleaner, which segregates older, more slowly changing data
from young rapidly-changing data and prioritizes cleaning older vs younger 

### Designing File Systems in 1990s

Main pieces of tech: processors, disks, main memory

* Processors getting faster
* Disks getting cheaper, *but* not faster (hard to optimize seek time )
* Main memory getting way bigger/cheaper, therby absorbing a lot of the read requests and allowing their use as write buffers

Problems with current files systems

* too many small accesses 
* write synchronously (b/c dominated by writes to meta-data structures)

**LFS**

* Buffer writes, turning them async
* Issues
  1. retrieve info from log
  2. how to ensure free data 

Addressing 1

* Use i-node map that maintains current location of i-nodes; given the identifying number for a file, the i-node map must be indexed to determine the disk address of the i-node.

  Can fit in main memory (yay)

Addressing 2

Segment cleaning: copying live data out of a segment 