---
title: "kubernetes install"
collection: teaching
type: "技术"
excerpt: ''
permalink: /teaching/k8s-install
date: 2025-07-05
---

# 安装环境

virtual box 进行节点虚拟

master 2c6G
node1 2c4G
node2 2c4G
jumper 2c4G

# k8s安装

  按照大佬的文档一步一步走即可。
  https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/master


# 遇到的问题

## 镜像访问的问题。

如果您在 /etc/containerd/config.toml 中没有找到 [plugins."io.containerd.grpc.v1.cri".registry] 这部分内容，这通常意味着您的 Containerd 配置文件是一个较旧的版本，或者配置是默认的且未展开。

在较新版本的 Containerd 中，registry 相关的配置通常被独立出来，放在 /etc/containerd/certs.d/ 目录下。不过，您看到的是 [plugins."io.containerd.grpc.v1.cri".registry] 没有，这说明可能需要添加整个 plugins 块。

### 解决方案步骤:

#### 步骤 1：备份现有配置文件

在进行任何修改之前，始终建议备份原始配置文件：

``` bash

    sudo cp /etc/containerd/config.toml /etc/containerd/config.toml.bak

```

#### 步骤 2：生成一个默认的 Containerd 配置

如果您的 config.toml 文件非常简短或者缺失大部分内容，最安全的方法是生成一个带有所有默认设置的完整配置文件，然后在此基础上进行修改。

``` bash

    sudo containerd config default | sudo tee /etc/containerd/config.toml

```

这个命令会生成一个包含所有默认配置的 config.toml 文件。生成后，您可以再次打开它：

``` bash

sudo nano /etc/containerd/config.toml

```

现在，您应该能找到 [plugins."io.containerd.cri.v1.images".registry] 这一段了。

#### 步骤 3：添加或修改 registry.mirrors 配置

找到[plugins."io.containerd.cri.v1.images".registry]部分后，在其内部添加或修改 mirrors 配置。您需要特别关注为 registry.k8s.io 和 docker.io 配置镜像。

在 config.toml 文件中，找到或者添加以下结构：

``` toml
    [plugins."io.containerd.cri.v1.images"]
  [plugins."io.containerd.cri.v1.images".registry]
    [plugins."io.containerd.cri.v1.images".registry.mirrors]
      # 为 Docker Hub 配置镜像加速器（如果需要）
      [plugins."io.containerd.cri.v1.images".registry.mirrors."docker.io"]
        endpoint = [""] # 或其他公共加速器

      # 关键：为 registry.k8s.io 配置镜像加速器
      [plugins."io.containerd.cri.v1.images".registry.mirrors."registry.k8s.io"]
        endpoint = ["https://k8s.m.daocloud.io"] # 替换为您的实际阿里云加速器地址
      
      # 如果您的集群仍在使用 k8s.gcr.io，也请添加
      [plugins."io.containerd.cri.v1.images".registry.mirrors."k8s.gcr.io"]
        endpoint = ["https://k8s-gcr.m.daocloud.io"]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"] # 配置 docker.io 的镜像加速器
        endpoint = ["https://docker.m.daocloud.io"]
```

#### 步骤 4：重启 Containerd 服务

保存 config.toml 文件后，重启 Containerd 服务以使更改生效：

``` bash

sudo systemctl restart containerd

```


#### 步骤 5：验证修复

##### 检查 Containerd 日志：

``` bash

sudo journalctl -u containerd -f

```

##### 手动拉取 pause 镜像：

``` bash

sudo crictl pull registry.k8s.io/pause:3.10

```


##### 重新创建 Pod：

删除之前卡住的 Pod，让 Kubernetes 重新调度和创建它。

``` bash

kubectl delete pod <问题Pod名称> -n <命名空间>

```

### 镜像源查找

https://github.com/DaoCloud/public-image-mirror  试了那么多家，就发现daocloud 靠谱，