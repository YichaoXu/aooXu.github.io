---
title: "Access Control"
date: "2020-04-26"
tags: ["seedlab", "lecture", "os-kernel"]
---

# 什么是访问控制

根据Prof. Anderson在Security Engineering中的描述, 访问控制的主要功能是去控制那些主体(principals)能够访问那些系统资源(resources). 所以我认为具体实现在OS上的访问控制（Access Control）应该是一组机制（Mechanisms）, 例如在Unix和Win上实现的用户组, 权限, 文件访问构成了OS上的访问控制. 但是在设计这些机制描述原理时, 我们所指的访问控制应该是这些机制实现的Policies, 而我们所说的访问控制模型则是这些Policies的Pattern.

# 总结

* 在描述具体OS中的AC时, 它表示一系列Mechanisms. 
* 在描述AC的设计和原理是, 它表示一系列Policies, 
* 在说到AC Model时, 它表示Policies的设计模式. 
