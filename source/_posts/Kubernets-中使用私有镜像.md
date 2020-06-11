---
title: Kubernets 中使用私有镜像
category: 思考
tags: 笔记
date: 2018-06-11 17:25:01
img:
---
```bash
Error response from daemon: pull access denied for xxx repository does not exist or may require 'docker login': denied: requested access to the resource is denied 
```

这是由于访问私有仓库时需要凭证。
<!--more-->

### 创建 Secret 存取 docker credentials 信息

> in the case of docker, only DockerConfig type secrets are honored. 

用于docker credentials 的 Secret 和普通的 Secret不一样，有个特殊类型 `Type:  kubernetes.io/dockerconfigjson`

#### 基于已经存在的Docker凭据创建一个Secret

```bash
# config.json
$ cat /root/.docker/config.json
{
        "auths": {
                "harbor.ainnovation.com": {
                        "auth": "eWltaW5qaWFuZzptdGF4QE1MSTIwMTk="
                }
        },
        "HttpHeaders": {
                "User-Agent": "Docker-Client/19.03.8 (linux)"
        }
}#

# create
$ kubectl create secret generic docker-creds-from-config --from-file=.dockerconfigjson=/root/.doc
ker/config.json --type=kubernetes.io/dockerconfigjson
secret/docker-creds-from-config created

# describe
$ kubectl describe secret/docker-creds-from-config
Name:         docker-creds-from-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/dockerconfigjson

Data
====
.dockerconfigjson:  168 bytes

# get yaml
$ kubectl get secrets docker-creds-from-config -o yaml
apiVersion: v1
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSJoYXJib3IuYWlubm92YXRpb24uY29tIjogewoJCQkiYXV0aCI6ICJlV2x0YVc1cWFXRnVaenB0ZEdGNFFFMU1TVEl3TVRrPSIKCQl9Cgl9LAoJIkh0dHBIZWFkZXJzIjogewoJCSJVc2VyLUFnZW50IjogIkRvY2tlci1DbGllbnQvMTkuMDMuOCAobGludXgpIgoJfQp9
kind: Secret
metadata:
  creationTimestamp: "2020-06-10T09:23:13Z"
  name: docker-creds-from-config
  namespace: default
  resourceVersion: "9108856"
  selfLink: /api/v1/namespaces/default/secrets/docker-creds-from-config
  uid: 06eb89a7-65e1-4ee7-b0cf-66d231a9c3de
type: kubernetes.io/dockerconfigjson

# 为了更好地理解.dockerconfigjson 字段，我们将它格式化一下
$ kubectl get secrets docker-creds-from-config -o "jsonpath={.data.\.dockerconfigjson}" | base64 --decode
{
        "auths": {
                "harbor.ainnovation.com": {
                        "auth": "eWltaW5qaWFuZzptdGF4QE1MSTIwMTk="
                }
        },
        "HttpHeaders": {
                "User-Agent": "Docker-Client/19.03.8 (linux)"
        }
}#
# 解析 auth
echo eWltaW5qaWFuZzptdGF4QE1MSTIwMTk= | base64 --decode
photon:pass4Photon
```



#### 通过在命令行中提供凭据来创建一个Secret

```bash
# create
$ kubectl create secret docker-registry docker-creds-from-literal --docker-server=harbor.ainnovation.com --docker-username=photon --docker-password=pass4Photon --docker-email=photon@ainnovation.com
secret/docker-creds-from-literal created

# describe
$ kubectl describe secret/docker-creds-from-literal
Name:         docker-creds-from-literal
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/dockerconfigjson

Data
====
.dockerconfigjson:  150 bytes

# get yaml
$ kubectl get secret/docker-creds-from-literal -o yaml
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJoYXJib3IuYWlubm92YXRpb24uY29tIjp7InVzZXJuYW1lIjoicGhvdG9uIiwicGFzc3dvcmQiOiJwYXNzNFBob3RvbiIsImVtYWlsIjoicGhvdG9uQGFpbm5vdmF0aW9uLmNvbSIsImF1dGgiOiJjR2h2ZEc5dU9uQmhjM00wVUdodmRHOXUifX19
kind: Secret
metadata:
  creationTimestamp: "2020-06-10T09:27:55Z"
  name: docker-creds-from-literal
  namespace: default
  resourceVersion: "9111178"
  selfLink: /api/v1/namespaces/default/secrets/docker-creds-from-literal
  uid: 881b08fd-5c4d-4d8b-b118-e8a8aafdf6f5
type: kubernetes.io/dockerconfigjson

# 为了更好地理解.dockerconfigjson 字段，我们将它格式化一下
$ kubectl get secret/docker-creds-from-literal -o "jsonpath={.data.\.dockerconfigjson}" | base64 --decode
{"auths":{"harbor.ainnovation.com":{"username":"photon","password":"pass4Photon","email":"photon@ainnovation.com","auth":"cGhvdG9uOnBhc3M0UGhvdG9u"}}}#


```



### Pod 中使用 Secret 完成docker login

`test-docker-creds.yaml`

```yaml
apiVersion: v1
 kind: Pod
 metadata:
   name: test-docker-creds
 spec:
   containers:
   - name: test-docker-creds
     image: harbor.ainnovation.com/runtime/jupyter:latest
   imagePullSecrets:
   - name: docker-creds-from-literal
```

创建

```bash
# create
$ kubectl create -f test-docker-creds.yaml
pod/test-docker-creds created
```



### 注意事项

* 用于docker credentials 的 Secret 和普通的 Secret不一样，有个特殊类型 `Type:  kubernetes.io/dockerconfigjson`
* Secret 是区分 namespace的

```bash
$ kubectl api-resources
NAME                              SHORTNAMES          APIGROUP                           NAMESPACED   KIND
secrets                                                                                  true         Secret
```

