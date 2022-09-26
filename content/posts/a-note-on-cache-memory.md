---
layout: post
title: "A Note on Cache Memory"
date: 2021-03-02
tags: [cache, draft]
draft: true
---

This is my lecture note of the YouTube course [Cache Memories](https://www.youtube.com/playlist?list=PLbtzT1TYeoMgJ4NcWFuXpnF24fsiaOdGq) by Prof. [Harry H. Porter III](http://web.cecs.pdx.edu/~harry/).

<!--more-->

## Lecture 1/9: Intro
### Concept
Memory (DRAM) is slow and cheap, cache (SRAM) is expensive and fast. Some data is frequently used, most data is not. So we keep those important data in cache.

![](https://i.imgur.com/uN8yOT0.png)

### Common Units of Cache
Unit size of a cache line (of block size) is 64 bytes. This is the unit size that a cache update itself, or a chunk of data transferred.

L1 cache total size ~ 32 KB, 4 cycles to access
L2 cache total size ~ 256 KB, 11 cycles to access
L3 cache total size ~ 8 MB, 30 cycles to access
Main memory ~ 16 GB, 100 cycles to access

## Lecture 2/9: Basic Concepts

## Lecture 3/9: Locality
Typical program spend a long time only for small amount of instructions, these are called working set. The instruction can also be cached. After a while, the program goes on and do another working set.

The data in the working set is heavily used. Caching those data could also improve performance.

### Locality

Both applies to data and instruction.

- Temporal locality: a byte that is used recently is likely to be used again soon
- Spatial locality: a byte is that is used recently, its nearby bytes are likely to be used soon


## Lecture 4/9: Writes and Coherency

### read
- Cache hit: data store in cache block, return the data very fast
- Cache miss: data not in cache block, read it from memory, store in cache and return the data, also evit a block (by least recently used)

### write
- Write hit:
    - write through: immediately writes modified block to memroy
    - **wirte back**: writes block to memroy when the block is evited, need a dirty bit to note the block has been modified (dirty bit == 1)
- Write miss:
    - **write allocate**: read into cache, update in the cache, set dirty bit to 1
    - write no allocate: send the write through to memroy, do not load block into cache

### Cache Line
Suppose we have a 64 bytes cache block, 64-bit address. The cache line would look like this.

![](https://i.imgur.com/FUgBkG6.png)

The address part only need 58-bit. It indicates the beginning of 64-bytes data in memory.
The dirty bit denotes this cache has been modified (1) or not (0).
The valid bit denotes this cache block contains valid data (1) or not (0).

## Lecture 5/9: Associative, 6/9 Direct Mapped
Traditional memory: address goes in, data goes out.

Associative memory: Key goes in, data goes out.

The key is that for each input key, the memory needs to match it with all cache blocks' keys. The more the cache blocks the longer the lookup process will take.

Suppose we have this cache.

Block size: 64 bytes
Number of cache block: 512 lines
Size of the cache: 32 KB

### Fully Associative
Any block can go into any cache line.

Number of set: 1
Address size: 64 bits




### Direct Mapped
Each block can go into only one cache line.

