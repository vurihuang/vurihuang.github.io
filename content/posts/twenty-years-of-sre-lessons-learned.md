+++
title = "【译】Google SRE 20年的经验分享"
author = ["vuri"]
lastmod = 2023-11-23T03:39:56+08:00
tags = ["trans", "sre"]
draft = true
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [1. 故障越大缓解措施的风险就越大](#1-dot-故障越大缓解措施的风险就越大)
- [2. 在发生紧急情况之前尽可能进行故障演练](#2-dot-在发生紧急情况之前尽可能进行故障演练)
- [3. 所有变更都要灰度发布](#3-dot-所有变更都要灰度发布)
- [4. 需要有一个叫停一切的"按钮"](#4-dot-需要有一个叫停一切的-按钮)
- [5. 光有单元测试还不够 - 还需要集成测试](#5-dot-光有单元测试还不够-还需要集成测试)
- [6. 沟通群很重要!备用沟通群很重要!备用沟通群的备用群也很重要!!!](#6-dot-沟通群很重要-备用沟通群很重要-备用沟通群的备用群也很重要)
- [7. Intentionally degrade performance modes](#7-dot-intentionally-degrade-performance-modes)
- [8. Test for Disaster resilience](#8-dot-test-for-disaster-resilience)
- [9. Automate your mitigations](#9-dot-automate-your-mitigations)
- [10. Reduce the time between rollouts, to decrease the likelihood of the rollout going wrong](#10-dot-reduce-the-time-between-rollouts-to-decrease-the-likelihood-of-the-rollout-going-wrong)
- [11. A single global hardware version is a single point of failure](#11-dot-a-single-global-hardware-version-is-a-single-point-of-failure)

</div>
<!--endtoc-->

原文: [twenty years of sre lessons learned](https://sre.google/resources/practices-and-processes/twenty-years-of-sre-lessons-learned/)

**作者**: Adrienne Walcer, Kavita Guliani, Mikel Ward, Sunny Hsiao, and Vrai Stacey

**贡献者**: Ali Biber, Guy Nadler, Luisa Fearnside, Thomas Holdschick, and Trevor Mattson-Hamilton

**前言**:

二十年可以发生很多事情,尤其是你在飞速成长的过程.
二十年前,Google 有一堆小型的数据中心,每个数据中心都有上千台服务器,它们之间听过2.4G的网络进行连接.我们通过一些 Python 脚本例如 "Assigner", "Autoreplacer" 和 "Babysitter" 诸如此类的来运行我们的私有云(虽然那个时候不是这么叫的),按照每个服务器的名称来划分配置来操作脚本.当时有个小型的数据库(MDB: Machines database)用来帮助我们长期组织管理维护每个服务器的信息.小团队里的工程师通过这些脚本和配置完成自动化的任务来解决一些常见的问题,并减轻人工管理机器集群的成本.


## 1. 故障越大缓解措施的风险就越大 {#1-dot-故障越大缓解措施的风险就越大}

> The riskiness of a mitigation should scale with the severity of the outage


## 2. 在发生紧急情况之前尽可能进行故障演练 {#2-dot-在发生紧急情况之前尽可能进行故障演练}

> Recovery mechanisms should be fully tested before an emergency


## 3. 所有变更都要灰度发布 {#3-dot-所有变更都要灰度发布}

> Canary all changes


## 4. 需要有一个叫停一切的"按钮" {#4-dot-需要有一个叫停一切的-按钮}

> Having a "Big Red Button"


## 5. 光有单元测试还不够 - 还需要集成测试 {#5-dot-光有单元测试还不够-还需要集成测试}

> Unit tests alone are not enough - integration testing is also needed


## 6. 沟通群很重要!备用沟通群很重要!备用沟通群的备用群也很重要!!! {#6-dot-沟通群很重要-备用沟通群很重要-备用沟通群的备用群也很重要}

> COMMUNICATION CHANNELS! AND BACKUP CHANNELS! AND BACKUPS FOR THOSE BACKUP CHANNELS!!!


## 7. Intentionally degrade performance modes {#7-dot-intentionally-degrade-performance-modes}


## 8. Test for Disaster resilience {#8-dot-test-for-disaster-resilience}


## 9. Automate your mitigations {#9-dot-automate-your-mitigations}


## 10. Reduce the time between rollouts, to decrease the likelihood of the rollout going wrong {#10-dot-reduce-the-time-between-rollouts-to-decrease-the-likelihood-of-the-rollout-going-wrong}


## 11. A single global hardware version is a single point of failure {#11-dot-a-single-global-hardware-version-is-a-single-point-of-failure}
