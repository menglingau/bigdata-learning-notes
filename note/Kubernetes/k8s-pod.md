




---
# 一、Pod基本概念


# 二、Pod存在的意义


# 三、Pod实现机制
1. 共享网络
    
    通过 Pause 容器, 把其他业务容器加入到 Pause 容器里去, 让所有业务容器在同一个名称空间, 可以实现网络共享.
2. 共享存储
    
    引入数据卷(Volumn)概念, 使用数据卷进行持久化存储
    
# 四、Pos镜像拉取策略
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec: 
  comtainers:
    - name: nginx
    inage: nginx:1.14
    imagePullPolicy: Always
```
imagePullPolicy有三个参数可选:
- IfNotPresent: 默认值, 镜像在宿主机上不存在时才拉取
- Always: 每次创建 pod 都会重新拉取一次镜像
- Never: Pod 永远不会主动拉取这个镜像


# 五、Pod资源限制
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  comtainers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
- resources.requests:  要求Container的内存和cpu必须有这么大
- resources.limits: 要求Container的内存最大为这么大

# 六、Pod重启机制
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: dns-test
spec:
  containers:
    - name: busybox
      image: busybox:1.28.4
      args:
      - /bin/sh
      - -c
      - sleep 36000
    restartPolicy: Never
```
restartPolicy有三个可选的参数:
- Always: 当容器终止退出后, 总是重启容器, 默认策略
- OnFailure: 当容器异常退出(退出状态码非0)时, 才重启容器
- Never: 当容器种植退出, 从不重启容器


# 七、Pod健康检查
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy
    livenessProbe(readinessProbe):
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
- livenessProbe(存活检查): 如果检查失败, 将杀死容器, 根据 Pod 的 restartPolicy 来操作.
- readinessProbe(就绪检查): 如果检查失败, k8s 会把该Pod从 service endpoints中剔除

Probe支持以下三种检查方式:
- httpGet: 发送 Http 请求, 返回200-400范围状态码为成功
- exec: 执行 Shell 命令返回状态码是0为成功
- tcpSocket: 发起 TCP Socket 建立成功

# 八、节点亲和性
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env_role
            operator: In
            values:
            - dev
            - test
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference: 
            matchExpressions:
            - key: group
              operator: In
              values:
              - otherprod
  containers:
  - name: webdemo
    image: nginx
```
- requiredDuringSchedulingIgnoredDuringExecution: 硬亲和性, 约束条件必须满足, 否则一直等待到条件满足
- preferredDuringSchedulingIgnoredDuringExecution: 软亲和性, 尝试满足, 不保证约束条件满足
  - weight: 权重
  - operator: 操作符
    - In
    - NotIn
    - Exists
    - Gt
    - Lt
    - DoesNotExists
    
# 九、label 对Pod调度的影响
[对节点创建 label](k8s-label.md)

```bash
spex: 
  nodeSelector"
    env_role: dev
  containers:
  - name: nginx
    image: nginx:1.15
```

# 十、污点和污点容忍
污点(Taint): 节点不做普通分配调度, 是node属性

和 `九、label 对Pod调度的影响`、 `八、节点亲和性` 类似

## 10.1 查看节点污染情况
```bash
$ kubectl describe node master | grep Taint
Taints:             node.kubernetes.io/unreachable:NoExecute
```
污点值有如下三个:
- NoSchedule: 一定不被调度
- PreferNoSchedule: 尽量不被调度
- NoExecute: 不会调度, 并且会驱除 Node 已有 Pod

## 10.2 为节点添加污点
```bash
# 添加污点
$ kubectl taint node k8s-master <key>=<value>:<effect>
$ kubectl taint node k8s-master kino=test:NoSchedule
node/k8s-master tainted

# 查看污点
$ kubectl describe node k8s-master | grep Taint
Taints:             kino=test:NoSchedule
```

## 10.3 污点容忍
```yaml
spec: 
  tolerations:
  - key: "ey"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: webdemo
    image: nginx
```