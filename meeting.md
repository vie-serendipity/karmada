# 场景

使用两种类型集群，idc+cloud，idc为本地集群，cloud为云上集群
在使用多集群hpa时，希望资源充足的情况下优先向idc调度，然后再向cloud调度

## 当前的方案

类似如下配置，优先尝试第一个affinity也就是idc，然后第二个affinity（idc和cloud）

```yaml
spec:
  autoScaling:
    ecsProvision: true
  placement:
    clusterAffinities:
    - affinityName: idc
      clusterNames:
      - idc
    - affinityName: cloud
      clusterNames:
      - idc
      - cloud
    replicaScheduling:
      replicaDivisionPreference: Weighted
      replicaSchedulingType: Divided
      weightPreference:
        dynamicWeight: AvailableReplicas
  resourceSelectors:
  - apiVersion: apps/v1
    kind: Deployment
    name: demo
  schedulerName: default-scheduler
```

### 问题

描述一下上述policy的调度过程

1. 部署工作负载和多集群hpa
2. 流量高峰期多集群hpa扩容时第一个affinity中idc集群资源不足，调度失败
3. 尝试调度第二个affinity，成功调度
4. 流量高峰期结束，多集群hpa缩容，但此时还是向第二个affinity调度，副本数并未回到idc集群

第二步中，idc集群和cloud集群的副本拆分是按照动态权重来的，预期还是希望优先idc集群再cloud集群

第四步中，当多集群hpa缩容后，副本不会回到idc集群

## 解决方案
