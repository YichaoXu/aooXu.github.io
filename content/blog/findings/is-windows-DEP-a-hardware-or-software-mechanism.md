---
title: "is Windows DEP a hardware or software mechanism"
date: "2020-04-25"
tags: ["seedlab", "lecture", "os-kernel"]
---

# 1. 什么是DEP

DEP就是Data Execution Protection, 它是微软的一种基于page机制的数据保护机制(其他的一些操作系统也采取了这样的策略). 在面对计算机内存的攻击中, 存在一种常见的方式就是在内存中某一段将恶意代码以数据的形式输入, 在使用某些攻击方式执行这一部分代码, 实现期望的目的. 而DEP的保护原理就是NX(No Execution), 就是说将内存中的数据页标记为不能执行, 在任何时候任何程序想要执行数据页中的代码都是不可能的

# 2. DEP的实现

## 2.1 NX bits 

CPU中为每一Page添加一个额外的NX bits, 用来标记某一页是否能够执行, 属于硬件的实现方式. 被现在的大部分CPU实现. 

## 2.2 Emulated

通过操作系统中的Segmentation来进行模拟, 即使CPU提供硬件上的支持, 也能实现No Execution, 但是存在性能上的问题. 

# 3. 为什么WIN OS上的安全机制需要硬件上的实现

因为现代的DEP机制被大部分OS采用, 但是通过OS的模拟实现的方式对CPU的性能有所损耗. 所以CPU制造商通过在CPU中加入NXbit实现来提高性能. 
