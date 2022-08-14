
节点亲和性 是 Pod 的一种属性，它使 Pod 被吸引到一类特定的节点 （这可能出于一种偏好，也可能是硬性要求）。 污点（Taint） 则相反——它使节点能够排斥一类特定的 Pod。

容忍度（Toleration） 是应用于 Pod 上的。容忍度允许调度器调度带有对应污点的 Pod。 容忍度允许调度但并不保证调度：作为其功能的一部分， 调度器也会评估其他参数。

污点和容忍度（Toleration）相互配合，可以用来避免 Pod 被分配到不合适的节点上。 每个节点上都可以应用一个或多个污点，这表示对于那些不能容忍这些污点的 Pod， 是不会被该节点接受的


```bash
[root@k8s-master01 ~]# kubectl taint node k8s-node01 taintKey=taintValue:NoSchedule
node/k8s-node01 tainted

[root@k8s-master01 ~]# kubectl describe node k8s-node01 | grep taint
Taints:             taintKey=taintValue:NoSchedule

[root@k8s-master01 ~]# kubectl taint node k8s-node01 taintKey=taintValue:NoSchedule-
node/k8s-node01 untainted
[root@k8s-master01 ~]# kubectl describe node k8s-node01 | grep taint

```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
```

通过污点和容忍度，可以灵活地让 Pod 避开某些节点或者将 Pod 从某些节点驱逐。 下面是几个使用例子：

* `专用节点`：如果想将某些节点专门分配给特定的一组用户使用，你可以给这些节点添加一个污点（即， kubectl taint nodes nodename dedicated=groupName:NoSchedule）， 然后给这组用户的 Pod 添加一个相对应的容忍度 （通过编写一个自定义的准入控制器， 很容易就能做到）。 拥有上述容忍度的 Pod 就能够被调度到上述专用节点，同时也能够被调度到集群中的其它节点。 如果你希望这些 Pod 只能被调度到上述专用节点， 那么你还需要给这些专用节点另外添加一个和上述污点类似的 label （例如：dedicated=groupName）， 同时还要在上述准入控制器中给 Pod 增加节点亲和性要求，要求上述 Pod 只能被调度到添加了 dedicated=groupName 标签的节点上。

* `配备了特殊硬件的节点`：在部分节点配备了特殊硬件（比如 GPU）的集群中， 我们希望不需要这类硬件的 Pod 不要被调度到这些特殊节点，以便为后继需要这类硬件的 Pod 保留资源。 要达到这个目的，可以先给配备了特殊硬件的节点添加污点 （例如 kubectl taint nodes nodename special=true:NoSchedule 或 kubectl taint nodes nodename special=true:PreferNoSchedule)， 然后给使用了这类特殊硬件的 Pod 添加一个相匹配的容忍度。 和专用节点的例子类似，添加这个容忍度的最简单的方法是使用自定义 准入控制器。 比如，我们推荐使用扩展资源 来表示特殊硬件，给配置了特殊硬件的节点添加污点时包含扩展资源名称， 然后运行一个 ExtendedResourceToleration 准入控制器。此时，因为节点已经被设置污点了，没有对应容忍度的 Pod 不会被调度到这些节点。 但当你创建一个使用了扩展资源的 Pod 时，ExtendedResourceToleration 准入控制器会自动给 Pod 加上正确的容忍度，这样 Pod 就会被自动调度到这些配置了特殊硬件的节点上。 这种方式能够确保配置了特殊硬件的节点专门用于运行需要这些硬件的 Pod， 并且你无需手动给这些 Pod 添加容忍度。

* `基于污点的驱逐`: 这是在每个 Pod 中配置的在节点出现问题时的驱逐行为， 接下来的章节会描述这个特性。

前文提到过污点的效果值 NoExecute 会影响已经在节点上运行的 Pod，如下

 *   如果 Pod 不能忍受这类污点，Pod 会马上被驱逐。
 *   如果 Pod 能够忍受这类污点，但是在容忍度定义中没有指定 tolerationSeconds， 则 Pod 还会一直在这个节点上运行。
*  如果 Pod 能够忍受这类污点，而且指定了 tolerationSeconds， 则 Pod 还能在这个节点上继续运行这个指定的时间长度。

当某种条件为真时，节点控制器会自动给节点添加一个污点。当前内置的污点包括：

*    node.kubernetes.io/not-ready：节点未准备好。这相当于节点状况 Ready 的值为 "False"。
*    node.kubernetes.io/unreachable：节点控制器访问不到节点. 这相当于节点状况 Ready 的值为 "Unknown"。
*    node.kubernetes.io/memory-pressure：节点存在内存压力。
*    node.kubernetes.io/disk-pressure：节点存在磁盘压力。
*    node.kubernetes.io/pid-pressure: 节点的 PID 压力。
*    node.kubernetes.io/network-unavailable：节点网络不可用。
*   node.kubernetes.io/unschedulable: 节点不可调度。
*    node.cloudprovider.kubernetes.io/uninitialized：如果 kubelet 启动时指定了一个“外部”云平台驱动， 它将给当前节点添加一个污点将其标志为不可用。在 cloud-controller-manager 的一个控制器初始化这个节点后，kubelet 将删除这个污点。

在节点被驱逐时，节点控制器或者 kubelet 会添加带有 NoExecute 效果的相关污点。 如果异常状态恢复正常，kubelet 或节点控制器能够移除相关的污点。

