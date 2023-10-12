+++
title = "Kubectl 速查表"
author = ["vuri"]
lastmod = 2023-10-13T01:26:08+08:00
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [Node](#node)
    - [查看 Node 资源使用情况](#查看-node-资源使用情况)
    - [获取指定Node上的pod列表](#获取指定node上的pod列表)
    - [获取节点总可用资源](#获取节点总可用资源)
    - [节点更新维护](#节点更新维护)
- [Pod](#pod)
    - [设置Pod 调度策略](#设置pod-调度策略)

</div>
<!--endtoc-->


## Node {#node}


### 查看 Node 资源使用情况 {#查看-node-资源使用情况}

```shell
$ kubectl top node
NAME                    CPU(cores)   CPU%        MEMORY(bytes)   MEMORY%
172.16.192.36           1961m        12%         16885Mi         61%
172.16.193.199          839m         5%          8046Mi          27%
172.16.193.75           1915m        12%         10597Mi         38%
```


### 获取指定Node上的pod列表 {#获取指定node上的pod列表}

```shell
$ kubectl get po -A -o wide --field-selector spec.nodeName=172.16.192.36
NAMESPACE     NAME                   READY    STATUS    RESTARTS   AGE     IP               NODE            NOMINATED NODE   READINESS GATES
kube-system   kube-proxy-6gbzq       1/1      Running   0          13d     172.16.192.36    172.16.192.36   <none>           <none>
kube-system   node-local-dns-tqpkz   1/1      Running   0          2d7h    172.16.192.36    172.16.192.36   <none>           <none>
kube-system   kube-proxy-6gbzq       1/1      Running   0          13d     172.16.192.36    172.16.192.36   <none>           <none>
```


### 获取节点总可用资源 {#获取节点总可用资源}

```shell
$ kubectl get node -o=custom-columns="NODE:.metadata.name,ALLOCATABLE CPU:.status.allocatable.cpu,ALLOCATABLE MEMORY:.status.allocatable.memory"
NODE                    ALLOCATABLE CPU   ALLOCATABLE MEMORY
172.16.192.36           15600m            28262728Ki
172.16.193.199          15890m            30121540Ki
172.16.193.75           15600m            28262728Ki
```


### 节点更新维护 {#节点更新维护}

```shell
# 1. 标记节点不可被调度
$ kubectl cordon 172.16.192.36

# 2.驱逐节点上的所有 pod（除daemonset），并删除临时盘
$ kubectl drain 172.16.192.36 --ignore-daemonsets --delete-emptydir-data

# 3. 重新标记节点为可调度
$ kubectl uncordon 172.16.192.36
```


## Pod {#pod}


### 设置Pod 调度策略 {#设置pod-调度策略}

硬亲和（ `requiredDuringSchedulingIngoredExecution` ）：

```yaml
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

软亲和（ `preferredDuringSchedulingIgnoredExecution` ）

```yaml
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
