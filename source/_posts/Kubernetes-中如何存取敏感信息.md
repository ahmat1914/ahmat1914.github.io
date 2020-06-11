---
title: Kubernetes 中如何存取敏感信息
category: 思考
tags: 笔记
date: 2018-06-11 17:25:22
img:
---

### 什么是 Secret 对象

Secret是用来保存和传递密码、密钥、认证凭证这些敏感信息的对象。

这种方式的好处包括：意图明确，避免重复，减少暴露机会。
<!--more-->

####  存取敏感信息（sensitive data）常见的方式有：

* 将 Secret 对象当环境变量(environment variables)使用
* 將 Secrets File 掛載(mount) 在 Pod 某個檔案路徑底下使用
* 將這些 sensitive data 統一存放在某一個 Docker Image 中，並將這個 Image 存放在`私有的 Image Registry` 中，透過 `image pull` 下載到 Kubernetes Cluster 中，讓其他 Pods 存取。

### 实战：Kubernetes 中如何创建 Secret 对象？

### 从文件输入创建 Secret

```bash
# files
$ echo -n "root" > ./username.txt
$ echo -n "rootpass" > ./password.txt

# create
$ kubectl create secret generic test-secret-from-file --from-file=./username.txt --from-file=./password.txt
secret/test-secret-from-file created

# describe
$ k describe secret/test-secret-from-file
Name:         test-secret-from-file
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password.txt:  8 bytes
username.txt:  4 bytes

```



#### 从命令行输入创建 Secret

```bash
# create
$ kubectl create secret generic test-secret-from-literal --from-literal=username=root --from-literal=p
assword=rootpass
secret/test-secret-from-literal created

# describe
$ kubectl describe secret/test-secret-from-literal
Name:         test-secret-from-literal
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  8 bytes
username:  4 bytes
```

#### 从 YAML 定义输入创建 Secret

首先，必须把敏感信息值用 [base64 編碼](https://zh.wikipedia.org/wiki/Base64) ，以 **username=root, password=rootpass** 为例，

```bash
$ tmp echo -n "root" | base64
cm9vdA==
$ tmp echo -n "rootpass" | base64
cm9vdHBhc3M=
```

把编码过的值写入 YAML 定义中，如 `test-secret-from-yaml.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret-from-yaml
type: Opaque
data:
  username: cm9vdA==
  password: cm9vdHBhc3M=
```

创建

```bash
# create
$ kubectl create -f test-secret-from-yaml.yaml
secret/test-secret-from-yaml created

# describe
$ kubectl describe secret/test-secret-from-yaml
Name:         test-secret-from-yaml
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  8 bytes
username:  4 bytes
```



### 实战：Pods 中如何使用 Secret 对象？

#### 传给环境变量（`environment variables`）

`test-secret-pod.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-secret-pod
  labels:
    app: webserver
spec:
  containers:
  - name: test-secret-pod
    image: zxcvbnius/docker-demo
    ports:
    - containerPort: 3000
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: test-secret-from-yaml
          key: username
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: test-secret-from-yaml
          key: password
```

**若该 Secret 对象是从 `--from-file` 创建，Kubernetes 会找 key 对应的文件，取出值作为 value**。

创建

```bash
# create
$ kubectl create -f test-secret-pod.yaml
pod/test-secret-pod created

# describe
$ kubectl describe pod/test-secret-pod
...
Environment:
      SECRET_USERNAME:  <set to the key 'username' in secret 'test-secret-from-yaml'>  Optional: false
      SECRET_PASSWORD:  <set to the key 'password' in secret 'test-secret-from-yaml'>  Optional: false
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-4jvjd (ro)
...

# echo envs
$ kubectl exec -it pod/test-secret-pod -- /bin/bash
root@test-secret-pod:/app# echo $SECRET_USERNAME
root
root@test-secret-pod:/app# echo $SECRET_PASSWORD
rootpass
```

#### [Secrets File](https://kubernetes.io/docs/concepts/configuration/secret/) 挂载到 Pod 中某个目录

`test-secret-pod-mount.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-secret-pod-mount
  labels:
    app: webserver
spec:
  containers:
  - name: test-secret-pod-mount
    image: zxcvbnius/docker-demo
    ports:
    - containerPort: 3000
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/creds
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: test-secret-from-yaml
```

创建

```bash
# create
$ kubectl create -f test-secret-pod-mount.yaml
pod/test-secret-pod-mount created

# describe
$ kubectl describe pod/test-secret-pod-mount
...
Environment:    <none>
    Mounts:
      /etc/creds from secret-volume (ro)
...
Volumes:
  secret-volume:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  test-secret-from-yaml
    Optional:    false
...
# check mounted files
$ kubectl exec -it test-secret-pod-mount -- /bin/bash
root@test-secret-pod-mount:~# cat /etc/creds/username
root

root@test-secret-pod-mount:~# cat /etc/creds/password
rootpass
```

> 需要注意的是，敏感信息一旦存入 Secret 对象中，其他人也可以在 Kubernetes Cluster 中存取这些敏感信息。因此，需要 Secret和 ServiceAccount搭配使用。
