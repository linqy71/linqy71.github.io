---
title: Lightweight and Accurate Memory Allocation in Key-Value Cache
date: 2021-02-25 20:16:57
tags: key-value cache, memcached
categories: 论文阅读
copyright:
---

# Abstract

- 影响memcached性能的因素：memory allocation among different item classes.
- 本文提出了一种用于memcached的轻量的、准确的内存分配模式。
- 早期的内存分配优化工作：LAMA，使用了基于footprint的MRC（miss ratio curves），但没有考虑memcached的删除操作，且空间开销非常大。
- 相比于LAMA，本文提出的方法的空间开销为LAMA的3%，能够处理读、写、删除操作，且miss rate降低了15%、平均响应时间减少了12.3%。

# Introduction

- memcached，最常用的分布式内存键值对缓存系统。
- memcached中的items根据大小被分为不同的class，而内存分配不考虑数据的locality。
- slab calcification: 当可用的slab被创建时，分配的空间无法再根据访问模式的改变而调整。
- LAMA（locality-aware memory allocation）
- 为了保持数据一致，写操作直接更新后端数据库，并将对应的key从memcached中删除。（？）

# MRC construction
- aet: average eviction time

# Lightweight and Accurate Memory Allocation
## Memory Allocation Controller


