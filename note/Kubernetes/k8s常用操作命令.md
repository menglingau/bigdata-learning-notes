
# get
① 查看节点状态
```bash
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS   ROLES    AGE    VERSION
k8s-master   Ready    master   2d4h   v1.18.0
k8s-node1    Ready    <none>   2d4h   v1.18.0
k8s-node2    Ready    <none>   2d4h   v1.18.0
```
② 查看指定 pod
```bash
[root@k8s-master yaml]# kubectl get pods -l app=nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5bf87f5f59-pkkrf   1/1     Running   0          4m14s
nginx-deployment-5bf87f5f59-sdvqd   1/1     Running   0          4m14s
```
`kubectl get` 指令的作用: 就是从 Kubernetes 里面获取（GET）指定的 API 对象

可以看到, 在这里我还加上了一个 -l 参数, 即获取所有匹配 app: nginx 标签的 Pod。

需要注意的是, 在命令行中, 所有 key-value 格式的参数, 都使用 “=” 而非 “:” 表示

③ 检查节点上各个系统 Pod 的状态
```bash
[root@k8s-master ~]# kubectl get pods -n kube-system
NAME                                 READY   STATUS    RESTARTS   AGE
coredns-7ff77c879f-7jvnh             1/1     Running   1          2d4h
coredns-7ff77c879f-kh7j9             1/1     Running   1          2d4h
etcd-k8s-master                      1/1     Running   1          2d4h
kube-apiserver-k8s-master            1/1     Running   1          2d4h
kube-controller-manager-k8s-master   1/1     Running   1          2d4h
kube-flannel-ds-9r9xv                1/1     Running   1          2d4h
kube-flannel-ds-n6x9j                1/1     Running   1          2d4h
kube-flannel-ds-wbjrv                1/1     Running   1          2d4h
kube-proxy-5lj6m                     1/1     Running   1          2d4h
kube-proxy-9n774                     1/1     Running   1          2d4h
kube-proxy-dv9r4                     1/1     Running   1          2d4h
kube-scheduler-k8s-master            1/1     Running   1          2d4h
```
④ 查看 deployment
```bash
[root@k8s-master yaml]# kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           2d5h
```


# describe
查看这个节点(Node)对象的详细信息、状态和事件(Event)
```bash
[root@k8s-master ~]# kubectl describe node k8s-master
Name:               k8s-master
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
```


# create
语法:
```bash
[root@k8s-master ~]# kubectl create -f yaml文件名
```
示例:
```bash
# create 
[root@k8s-master ~]# kubectl create -f nginx-deployment.yaml

# kubectl get 命令检查这个 YAML 运行起来的状态是不是与我们预期的一致
[root@k8s-master yaml]# kubectl get pods -l app=nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5bf87f5f59-pkkrf   1/1     Running   0          4m14s
nginx-deployment-5bf87f5f59-sdvqd   1/1     Running   0          4m14s
```
nginx-deployment.yaml
```yaml
[root@k8s-master ~]# vim nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

# describe
查看 node、pod 等详细信息
```bash
# 查看 node 详细信息
[root@k8s-master yaml]# kubectl describe node k8s-node1

# 查看 pod 详细信息
# 查看有哪些 pod
[root@k8s-master yaml]# kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5bf87f5f59-pkkrf   1/1     Running   0          9m40s
nginx-deployment-5bf87f5f59-sdvqd   1/1     Running   0          9m40s
# 查看指定 pod 的详细信息
[root@k8s-master yaml]# kubectl describe pod nginx-deployment-5bf87f5f59-pkkrf
```
在 `kubectl describe` 命令返回的结果中, 你可以清楚地看到这个 Pod 的详细信息, 比如它的 IP 地址等等。

其中, 有一个部分值得你特别关注, 它就是 Events（事件）。

在 Kubernetes 执行的过程中, 对 API 对象的所有重要操作, 都会被记录在这个对象的 Events 里, 并且显示在 kubectl describe 指令返回的结果中。

比如, 对于这个 Pod, 我们可以看到它被创建之后, 被调度器调度（Successfully assigned）到了 node-1, 拉取了指定的镜像（pulling image）, 然后启动了 Pod 里定义的容器（Started container）。

所以, 这个部分正是我们将来进行 Debug 的重要依据。如果有异常发生, 你一定要第一时间查看这些 Events, 往往可以看到非常详细的错误信息。


# replace
更新替换资源(升级资源)

例如升级 Nginx(上面 crete 示例使用的是 Nginx1.7.9, 这里升级为1.8)
```yaml
[root@k8s-master ~]# vim nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        ports:
        - containerPort: 80

# 更新
[root@k8s-master ~]# kubectl replace -f nginx-deployment.yaml
deployment.apps/nginx-deployment replaced
```


# apply
apply 集合了 crete和replace两个指令的功能, 使用 `kubectl apply` kubernetes 会根据 YAML 文件的内容变化, 自动进行具体的处理。

例如上述 创建 Nginx、升级 Nginx的操作在这里可以写成:
```bash
[root@k8s-master ~]# kubectl apply -f nginx-deployment.yaml
[root@k8s-master ~]# vim nginx-deployment.yaml  # 更新版本
[root@k8s-master ~]# kubectl apply -f nginx-deployment.yaml
```

# exec
进入到指定的 pod 中
```bash
# 查看所有的 pod
[root@k8s-master yaml]# kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-9754ccbdf-5spwg   1/1     Running   0          2m12s
nginx-deployment-9754ccbdf-nlbwl   1/1     Running   0          87s

# 进入 pod
[root@k8s-master yaml]# kubectl exec -it nginx-deployment-9754ccbdf-5spwg -- /bin/bash
root@nginx-deployment-9754ccbdf-5spwg:/# ls /usr/share/nginx/html
```

# delete
① 根据yaml文件删除创建资源
```bash
[root@k8s-master yaml]# kubectl delete -f nginx-deployment.yaml

# 查看 pod
[root@k8s-master yaml]# kubectl get pods
NAME                               READY   STATUS        RESTARTS   AGE
nginx-deployment-9754ccbdf-5spwg   1/1     Terminating   0          4m50s
nginx-deployment-9754ccbdf-nlbwl   1/1     Terminating   0          4m5s

# 隔一会在查看 pod 
[root@k8s-master yaml]# kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE

```
② 删除已经创建的 pod
```bash
[root@k8s-master yaml]# kubectl delete pod nginx-f89759699-4tz6t
pod "nginx-f89759699-4tz6t" deleted
```
查看 pod 仍然存在
```bash
[root@k8s-master yaml]# kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
nginx-f89759699-g7smv   1/1     Running   0          53s
```
删除deployment
```bash
# 查看 deployment
[root@k8s-master yaml]# kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           2d5h
[root@k8s-master yaml]# kubectl delete deployment nginx
deployment.apps "nginx" deleted
```
再次查看, pod 消失
```bash
[root@k8s-master yaml]# kubectl get pods
No resources found in default namespace.
```

# 一、污点相关
## 1.1 添加污点:
```bash
[root@k8s-master ~]# kubectl taint nodes <node-name> <key>=<value>:<effect>
[root@k8s-master ~]# kubectl taint nodes k8s-node1 foo=bar:NoSchedule
kubectl taint node k8s-node1 node-type=bar:NoSchedule
                     
```

## 1.2 查看污点
① 方式一
```bash
[root@k8s-master ~]# kubectl get nodes <nodename> -o go-template={{.spec.taints}}
[root@k8s-master ~]# kubectl get nodes k8s-node1 -o go-template={{.spec.taints}}
[map[effect:NoSchedule key:foo value:bar]]
```
② 方式二
```bash
[root@k8s-master ~]# kubectl describe node k8s-node1
Name:               k8s-node1
Roles:              <none>
...
Taints:             foo=bar:NoSchedule
```

## 1.3 删除污点
① 删除所有污点
```bash
[root@k8s-master ~]# kubectl patch nodes k8s-node1 -p '{"spec":{"taints":[]}}'
```
② 删除指定key的污点
```bash
[root@k8s-master ~]# kubectl taint nodes --all foo-
[root@k8s-master ~]# kubectl taint nodes --all node-type-
```
③ 删除指定key, 指定value的污点
```bash
[root@k8s-master ~]# kubectl taint nodes --all foo:NoSchedule-
[root@k8s-master ~]# kubectl taint nodes --all node-type:NoSchedule-
```

## 1.4 污点容忍度
### ① 等值判断
```yaml
tolerations:
- key: "key1"
  operator: "Equal" #判断条件为 Equal
  value: "value1"
  effect: "NoExecute"
  tolerationSeconds: 3600
```
### ② 存在性判断
```yaml
tolerations: 
- key: "key1"
  operator: "Exists"#存在性判断，只要污点键存在，就可以匹配
  effect: "NoExecute"
  tolerationSeconds: 3600
apiVersion: v1
kind: Deployment
metadata: 
  name: myapp-deploy
  namespace: default
spec:
  replicas: 3
  selector: 
    matchLabels: 
      app: myapp
      release: canary
  template:
    metadata:
      labels:
        app: myapp
        release: canary
    spec:
      containers:
      - name: myapp
      image: ikubernetes/myapp:v1
      ports:
      - name:http
        containerPort: 80
      tolerations:
      - key:"node-type"
        operator: "Equal"
        value:"production":
        effect: "NoExecute"
        tolerationSeconds: 3600
```