---
title: "File Permission of Linux OS"
date: "2020-04-26"
tags: ["seedlab", "lecture", "os-kernel"]
---

# 1. 什么是DAC

DAC指自主访问控制(Discretionary Access Control), 是一种访问控制模式(Access Control Pattern). 在这种模式中主体(Principals)对于某个资源(resources)的权限不由系统决定而是由资源的创建者决定. 

# 2. 在Unix中的Access Control

## 2.1 Overview

Unix文件系统就采取了这种访问控制模式. 
其中主体对于资源都存在三种可能的行为: `read`, `write`和`execute`. 
而可能的资源包括目录和文件(directories and files), 进程(processes), 设备和管道(devices and named pipes).  

## 2.2 对每种资源的权限的解释

### 2.2.1 Files

读写和执行权限就是如字面上的含义

### 2.2.2 Directories

* Read: 有权限列出目录中的文件或者子目录
* Write: 添加或者删除其中文件或者子目录
* Exe: 指可以访问此路径, 并有同时访问其中文件权限的可能

### 2.2.3 Processes

* Read: 能够接受这个进程发生的信息 (signal). 
* Write: 能够向这个进程发生信息 (signal). 
* Exe: 能够调用这个进程作为一个子进程 (sub-process). 

### 2.2.4 Devices and Named pipe

* Read and Write: 和文件一样,
* Exe: 没有实际含义. 

# 3. 在常见Linux指令中的运用

## 3.1 ls -l 

ls -l 输出的结果是一个10位的String, 这个字符串的第一位是分隔符'-', 没有实际含义, 第2-4位表示创建者的权限, 第5-7位表示和创建者同一用户组的用户的权限, 第8-10位表示其他用户的权限. 例如如果输出的结果是`-rwxr-x---` 
* 第一个`-`没有实际含义
* 第2-4位`rwx`表示创建者的权限包括`r`ead, `w`rite 和e`x`ecute
* 第5-7位`r-x`表示创造者所处的用户组的权限是`r`ead, 和e`x`ecute
* 最后三位`---`表示其他人没有权限

## 3.2 Chmod

例如`chmod file 775`因为存在三种可能的操作, `read`, `write`和`execute`所以我们可以用三位二进制数字表示给予哪几种权限, 所以十进制的**7**就是二进制的**111**, 十进制的**5**就是二进制的**101**
