## The Design and Implementation of a Log-Structured File System

Published: 1991

[Link](https://people.eecs.berkeley.edu/~brewer/cs262/LFS.pdf)

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

1. read a number of segments into memory
2.  identify the live data
3.  write the live data back to a smaller number of clean segments.
4. mark cleaned segments as free 

Has segment summary info which holds file and block number

* To check if alive, check file's (or indirect block) i-node to see if appropriate block pointer still points to this block

Segment usage table keeps track of how many live bytes and most recent modified time of any block in segement

**Recovery**

* Uses checkpoints (point where log state is consistent) to restore during recovery

* Roll forward to see what data can be recovered after 

## A Fast File System for Unix

Published: 1984

[Link](https://dl.acm.org/doi/pdf/10.1145/989.990)

New reimplementation of the UNIX system with higher throughput rates by using more flexible allocation polices that allow better locality of references. Clusters data that is sequentially accessed and provides two block sizes. 

Achievements

* 10x faster file access time
* Add advisory locks, file locking, long file name support, and admin controls 

### Introduction

Problem: old file system cannot provide needed throughput; it could only provide about 2% of max disk bandwidth



Allocation of data blocks is sub-optimal: ``` The traditional file system never transfers more than 512 bytes per disk transaction and often finds that the next sequential data block is not on the same cylinder, forcing seeks between 512 byte transfers. ```

New File System Organization 

**Cylinder group**: one or more consecutive cylinders on a disk. 

*  Each disk drive contains one or more file systems
* File systems is described by its super block (located at beginning of file system's disk partition and replicated)
* Minimum block size is 4096 bytes
* Divides partition into cylinder groups
* Allows block to be split into addressable fragments for small files
* Adds systems parameters that govern behaviror 



**Global and local block allocation policy** 

Global calculates rotationally optimal block layout and tries to clusters related data and i-nodes of a file dir in the same cylinder group

Global calls local and asks for a block, if local can't give then

1. Use next block rotationally closest to the requested block on the same cylinder
2. Use a block within the same cylinder group
3. Quadratically hash the cylinder group number to choose another cylinder group to look for a free block
4. Apply an exhaustive search to all cylinder groups