---
{"dg-publish":true,"permalink":"/kubernetes/kubernetes/","title":"Kubernetes","created":"2024-02-28T15:03:46.010+08:00","updated":"2024-03-07T11:07:26.879+08:00"}
---

# 安装

## 环境

| 主机名 | IP | 操作系统 |
| :--: | :--: | :--: |
| master | 192.168.65.70 | Centos 7.9.2009 |
| node-01 | 192.168.65.71 | Centos 7.9.2009 |
| node-02 | 192.168.65.72 | Centos 7.9.2009 |

## 配置主机名

`master`
``` shell
hostnamectl set-hostname master
```

`node-01`
``` shell
hostnamectl set-hostname node-01
```

`node-02`
``` shell
hostnamectl set-hostname node-02
```

## 绑定host

`master、node-01、node-02`
``` shell
cat << EOF >> /etc/hosts
192.168.65.70 master
192.168.65.71 node-01
192.168.65.72 node-02
EOF
```

## 关闭防火墙

`master、node-01、node-02`
``` shell
systemctl stop firewalld
systemctl disable firewalld
```

## 禁用SELinux

`master、node-01、node-02`

`/etc/selinux/config`
![Pasted image 20240228163655.png](/img/user/images/Pasted%20image%2020240228163655.png)

> 将SELINUX设置为disabled

``` shell
setenforce 0
```

## 关闭swap交换分区

`master、node-01、node-02`

``` shell
swapoff -a
```

`/etc/fstab`
![Pasted image 20240228164610.png](/img/user/images/Pasted%20image%2020240228164610.png)

> 将swap这行注释掉

## Docker安装

`master、node-01、node-02`

[[Docker/Docker\|Docker]]

## 配置yum源

`master、node-01、node-02`
``` shell
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

## 安装kubelet、kubeadm、kubectl

`master、node-01、node-02`
``` shell
yum install -y kubelet kubeadm kubectl
```

> 如果出现索引 gpg 检查失败的情况，请使用一下命令

``` shell
yum install -y --nogpgcheck kubelet kubeadm kubectl
```

## 设置kubelet开机自启

`master、node-01、node-02`
``` shell
systemctl enable kubelet
```

## 内核优化

`master、node-01、node-02`

``` shell
cat << EOF >> /etc/sysctl.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

``` shell
sysctl -p
```

## 修改containerd仓库

`master、node-01、node-02`

``` shell
containerd config default > /etc/containerd/config.toml
```

`/etc/containerd/config.toml`
![Pasted image 20240229161454.png](/img/user/images/Pasted%20image%2020240229161454.png)

将`sandbox_image`修改为`registry.aliyuncs.com/google_containers/pause:3.6`

``` shell
systemctl daemon-reload
systemctl restart containerd
```

## 生成初始化配置文件

`master`
``` shell
kubeadm config print init-defaults > init-config.yaml
```

## 修改初始化配置文件

`master`

`init-config.yaml`
![Pasted image 20240228191211.png](/img/user/images/Pasted%20image%2020240228191211.png)

advertiseAddress：master节点IP地址
name：master节点主机名

![Pasted image 20240228191446.png](/img/user/images/Pasted%20image%2020240228191446.png)

imageRepository：修改为国内地址`registry.aliyuncs.com/google_containers`
podSubnet: 10.244.0.0/16 新增pod网段

## 拉取所需镜像

`master`
``` shell
kubeadm config images pull --config=init-config.yaml
```

如果出现`CRI v1 image API is not implemented for endpoint`的错误
`/etc/containerd/config.toml`

![Pasted image 20240228202913.png](/img/user/images/Pasted%20image%2020240228202913.png)

注释掉`disabled_plguins=["cri"]`即可

``` shell
systemctl restart containerd
```

## 初始化kubernetes

`master`
``` shell
kubeadm init --config=init-config.yaml
```

## 复制配置文件到$HOME目录

`master`
``` shell
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

## node节点加入集群

`node-01、node-02`

kubernetes初始化后会生成加入命令，例如
``` shell
kubeadm join 192.168.8.4:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:3a95cfb88f3951320621094e41eebe4e3b5d283312afe9613a131a6678d8123f
```

## 安装flannel网络

`master`
``` shell
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

如果下载不了可以手动创建

`kube-flannel.yml`
``` yml
---
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp.flannel.unprivileged
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: docker/default
    seccomp.security.alpha.kubernetes.io/defaultProfileName: docker/default
    apparmor.security.beta.kubernetes.io/allowedProfileNames: runtime/default
    apparmor.security.beta.kubernetes.io/defaultProfileName: runtime/default
spec:
  privileged: false
  volumes:
    - configMap
    - secret
    - emptyDir
    - hostPath
  allowedHostPaths:
    - pathPrefix: "/etc/cni/net.d"
    - pathPrefix: "/etc/kube-flannel"
    - pathPrefix: "/run/flannel"
  readOnlyRootFilesystem: false
  # Users and groups
  runAsUser:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  # Privilege Escalation
  allowPrivilegeEscalation: false
  defaultAllowPrivilegeEscalation: false
  # Capabilities
  allowedCapabilities: ['NET_ADMIN']
  defaultAddCapabilities: []
  requiredDropCapabilities: []
  # Host namespaces
  hostPID: false
  hostIPC: false
  hostNetwork: true
  hostPorts:
  - min: 0
    max: 65535
  # SELinux
  seLinux:
    # SELinux is unused in CaaSP
    rule: 'RunAsAny'
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: flannel
rules:
  - apiGroups: ['extensions']
    resources: ['podsecuritypolicies']
    verbs: ['use']
    resourceNames: ['psp.flannel.unprivileged']
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes/status
    verbs:
      - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: flannel
  namespace: kube-system
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
  labels:
    tier: node
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds-amd64
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/os
                    operator: In
                    values:
                      - linux
                  - key: kubernetes.io/arch
                    operator: In
                    values:
                      - amd64
      hostNetwork: true
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: registry.cn-zhangjiakou.aliyuncs.com/test-lab/coreos-flannel:amd64
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: registry.cn-zhangjiakou.aliyuncs.com/test-lab/coreos-flannel:amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
            add: ["NET_ADMIN"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run/flannel
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds-arm64
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/os
                    operator: In
                    values:
                      - linux
                  - key: kubernetes.io/arch
                    operator: In
                    values:
                      - arm64
      hostNetwork: true
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: registry.cn-zhangjiakou.aliyuncs.com/test-lab/coreos-flannel:arm64
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: registry.cn-zhangjiakou.aliyuncs.com/test-lab/coreos-flannel:arm64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
             add: ["NET_ADMIN"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run/flannel
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds-arm
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/os
                    operator: In
                    values:
                      - linux
                  - key: kubernetes.io/arch
                    operator: In
                    values:
                      - arm
      hostNetwork: true
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: registry.cn-zhangjiakou.aliyuncs.com/test-lab/coreos-flannel:arm
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: registry.cn-zhangjiakou.aliyuncs.com/test-lab/coreos-flannel:arm
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
             add: ["NET_ADMIN"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run/flannel
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds-ppc64le
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/os
                    operator: In
                    values:
                      - linux
                  - key: kubernetes.io/arch
                    operator: In
                    values:
                      - ppc64le
      hostNetwork: true
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: registry.cn-zhangjiakou.aliyuncs.com/test-lab/coreos-flannel:ppc64le
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: registry.cn-zhangjiakou.aliyuncs.com/test-lab/coreos-flannel:ppc64le
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
             add: ["NET_ADMIN"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run/flannel
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds-s390x
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/os
                    operator: In
                    values:
                      - linux
                  - key: kubernetes.io/arch
                    operator: In
                    values:
                      - s390x
      hostNetwork: true
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: registry.cn-zhangjiakou.aliyuncs.com/test-lab/coreos-flannel:s390x
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: registry.cn-zhangjiakou.aliyuncs.com/test-lab/coreos-flannel:s390x
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
             add: ["NET_ADMIN"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run/flannel
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
```

``` shell
kubectl apply -f kube-flannel.yml
```

# 命令

## 查看节点

``` shell
kubectl get nodes
```

## 查看节点详情

``` shell
kubectl get nodes -o wide
```

## 节点打标签

``` shell
kubectl label nodes <节点名称> labelName=<标签名称>
```

## 查看节点标签

``` shell
kubectl get node --show-labels
```

## 删除节点标签

``` shell
kubectl label node <节点名称> labelName
```

## 查看pod节点

``` shell
kubectl get pod
```

## 查看所有pod节点

``` shell
kubectl get pod -A
```

## 查看pod节点详情

``` shell
kubectl get pod -o wide
```

## 查看所有名称空间下的pod

``` shell
kubectl get pod --all-namespaces
```

## 根据yaml文件创建pod

``` shell
kubectl apply -f <文件名称>
```

## 根据yaml文件删除pod

``` shell
kubectl delete -f <文件名称>
```

## 删除pod节点

``` shell
kubectl delete pod <pod名称> -n <名称空间>
```

## 查看异常的pod节点

``` shell
kubectl get pods -n <名称空间> | grep -v Running
```

## 查看异常pod节点的日志

``` shell
kubectl describe pod <pod名称>  -n <名称空间>
```

## 进入默认命名空间的pod节点

``` shell
kubectl exec -it <pod名称> -- /bin/bash
```

## 进入某个特定命名空间下的pod节点

``` shell
kubectl exec -it <pod名称>  -n <命名空间> -- /bin/bash
```

## 普通方式创建pod

``` shell
kubectl run <pod名称> --image=<镜像名称>
```

## 监控pod(一秒钟更新一次命令)

``` shell
watch -n 1 kubectl get pod
```

## deployment部署pod(具有自愈能力，宕机自动拉起)  
  
``` shell
kubectl create deployment <pod名称> --image=<镜像名称>
```
  
## deployment部署pod(多副本)  
  
``` shell
kubectl create deployment <pod名称> --image=<镜像名称> --replicas=3
```
  
## 查看deployment部署  
  
``` shell
kubectl get deploy
```
  
## 删除deployment部署  
  
``` shell
kubectl delete deploy <pod名称>
```
  
## deployment扩容\缩容pod  
  
``` shell
kubectl scale deploy/<pod名称> --replicas=<5>
```
  
## deployment扩容\缩容pod  
  
``` shell
kubectl edit deploy <pod名称>
```
  
## deployment滚动更新pod  
  
``` shell
kubectl set image deploy/<pod名称> <容器名称>=<镜像名称:版本号> --record
```
  
## deployment查看pod回退版本  
  
``` shell
kubectl rollout history deploy/<pod名称>
```
  
## deployment查看pod回退版本详情  
  
``` shell
kubectl rollout history deploy/<pod名称> --revision=1
```
  
## deployment回退pod到上一个版本  
  
``` shell
kubectl rollout undo deploy/<pod名称>
```
  
## deployment回退pod到指定版本  
  
``` shell
kubectl rollout undo deploy/<pod名称> --to-revision=1
```
  
## deployment暴露pod集群内部访问（ClusterIP）  
  
``` shell
kubectl expose deploy <pod名称> --port=8080 --target-port=80 --type=ClusterIP
```
  
## deployment暴露pod外网访问（NodePort）  
  
``` shell
kubectl expose deploy <pod名称> --port=8080 --target-port=80 --type=NodePort
```

## 获取加入集群token

``` shell
kubeadm token create --print-join-command
```

## 查看名称空间

``` shell
kubectl get namespace
```

## 查看名称空间

``` shell
kubectl get ns
```

## 创建名称空间

``` shell
kubectl create ns <名称>
```

## 删除名称空间

``` shell
kubectl delete ns <名称>
```