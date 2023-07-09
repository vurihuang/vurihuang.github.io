---
layout: post
title: Kubectl 速查表
slug: kubectl-cheatsheet
---


<!-- vim-markdown-toc GFM -->

* [Node](#node)
  * [查看Node资源使用情况](#查看node资源使用情况)
  * [获取指定Node上的pod列表](#获取指定node上的pod列表)
  * [获取节点总可用资源](#获取节点总可用资源)
* [Pod](#pod)
  * [设置Pod 调度策略](#设置pod-调度策略)
    * [Pod 亲和性](#pod-亲和性)
      * [硬亲和（requiredDuringSchedulingIngoredExecution）](#硬亲和requiredduringschedulingingoredexecution)
      * [软亲和（preferredDuringSchedulingIgnoredExecution）](#软亲和preferredduringschedulingignoredexecution)
    * [Pod 反亲和性](#pod-反亲和性)

<!-- vim-markdown-toc -->

## Node

### 查看Node资源使用情况

``` sh
$ kubectl top node
NAME                    CPU(cores)   CPU%        MEMORY(bytes)   MEMORY%
172.16.192.36           1961m        12%         16885Mi         61%
172.16.193.199          839m         5%          8046Mi          27%
172.16.193.75           1915m        12%         10597Mi         38%
```

### 获取指定Node上的pod列表

``` sh
$ kubectl get po -A -o wide --field-selector spec.nodeName=172.16.192.36
NAMESPACE     NAME                   READY    STATUS    RESTARTS   AGE     IP               NODE            NOMINATED NODE   READINESS GATES
kube-system   kube-proxy-6gbzq       1/1      Running   0          13d     172.16.192.36    172.16.192.36   <none>           <none>
kube-system   node-local-dns-tqpkz   1/1      Running   0          2d7h    172.16.192.36    172.16.192.36   <none>           <none>
kube-system   kube-proxy-6gbzq       1/1      Running   0          13d     172.16.192.36    172.16.192.36   <none>           <none>
```

### 获取节点总可用资源

``` sh
$ kubectl get node -o=custom-columns="NODE:.metadata.name,ALLOCATABLE CPU:.status.allocatable.cpu,ALLOCATABLE MEMORY:.status.allocatable.memory"
NODE                    ALLOCATABLE CPU   ALLOCATABLE MEMORY
172.16.192.36           15600m            28262728Ki
172.16.193.199          15890m            30121540Ki
172.16.193.75           15600m            28262728Ki
```

## Pod

### 设置Pod 调度策略

#### Pod 亲和性

##### 硬亲和（requiredDuringSchedulingIngoredExecution）

``` yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
          - key: foo
            operator: In
            values:
            - bar
```

##### 软亲和（preferredDuringSchedulingIgnoredExecution）

``` yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - preference:
        matchExpressions:
        - key: foo
          operator: In
          values:
          - ""
      weight: 100
```

#### Pod 反亲和性
