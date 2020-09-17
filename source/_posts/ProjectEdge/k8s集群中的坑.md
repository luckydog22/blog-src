---
title: k8s集群中的坑
date: 2020-04-21 15:07:15
categories: 虚拟化/容器
tags:
- Kubernetes
- ProjectEdge
---

## 无法访问上游DNS服务器

修改`kube-system`命名空间的Config Map:`coredns`

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: coredns
  namespace: kube-system
  selfLink: ...
  uid: ...
  resourceVersion: ...
  creationTimestamp: ...
data:
  Corefile: |
    .:53 {
        errors
        ready
        health
        forward . 114.114.114.114
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
        }
        cache 30
        reload
        loadbalance
    }
```
