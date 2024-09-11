---
title: Hexagonal Architecture
date: 2024-09-11 17:04:10
tags: Traslation, Architecture
---

# 六边形架构

[原文](https://alistair.cockburn.us/hexagonal-architecture/
)

> 创建无需`UI`或数据库的应用，使其在数据库不可用时也能进行回归测试。

---

## 目的

让应用(application)能够被多端驱动：用户、程序、自动化测试和脚本等等。同时让程序的开发和测试独立于其生产环境的设备和数据库。

当驱动器(driver)想要在端口(port)处使用应用时，其需要发送一个请求。请求会被适配器(adapter)根据驱动器使用的方案转换成可用的过程调用或消息，然后再传递到应用的端口处。应用可以完全的忽视驱动器使用的方案。
