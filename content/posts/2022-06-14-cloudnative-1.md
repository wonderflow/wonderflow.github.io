---
title: "云原生技术（1） - 如何从代码到制作并发布一个 Helm 包？"
date: 2022-06-14T14:02:05+08:00
---

## Helm 是什么？

云原生领域应用打包和分发的**事实标准**，Helm Chart 通常包含 Docker 镜像及其基础设施配置，能够把一个 K8s 生态的应用完整封装，并且在另一个 K8s 环境正常的运行。

## 为什么 Helm 会流行？

核心功能有两点：

1. 对复杂的 Kubernetes YAML 做了打包和抽象，简化为少量参数。
2. 给出应用的概念，并给出了完整生命周期解决方案：制作、上传（托管）、版本化、分发、部署。

真正让他流行起来的原因是：

3. 踩准了时机，当时（2018年） K8s 生态对 YAML 深恶痛绝但是苦于没有好的工具，快速形成了丰富的生态，如今已经有 1000+ 开箱即用的 Helm Chart: https://artifacthub.io/


## Helm Chart 解决了云原生应用交付所有问题了吗？

没有，这个问题以后会逐步回答，今天的重点是怎么玩好 Helm。


## 如何从源代码开始制作一个 Helm 包？

### 准备工具

- git
- docker
- helm
- velad
- kubectl

### 从代码到容器镜像

Helm Chart 是对 Kubernetes 资源的打包，所以制作 Helm Chart 的前提是需要对 Kubernetes 的[常用对象](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)和容器镜像有基本的了解。如果你对这些概念不熟悉，可以直接跳转到[如何部署 Helm 包](#如何部署-helm-包) 一节。

1. 准备好你的代码，比如一个 [2048 小游戏](https://github.com/gabrielecirulli/2048)：

```
git clone https://github.com/gabrielecirulli/2048.git
cd 2048
```

2. 将代码构建成容器镜像，构建镜像需要编写一个 [Dockerfile](https://docs.docker.com/engine/reference/builder/)，如果你对 Dockerfile 不太熟悉，没关系，对于所有**前端项目**而言，下面的 Dockerfile 基本是万能的，可以直接复用。

把以下内容复制到一个叫 `Dockerfile` 的文件中。

```
FROM nginx:latest
COPY . /usr/share/nginx/html
```

现在，我们的代码目录如下：
```
$ tree
.
├── CONTRIBUTING.md
├── Dockerfile
├── README.md
...
└── style
...

4 directories, 33 files
```

3. 编译镜像

```
docker build -t 2048 .
```

4. 本地试运行

```
docker run -p 8080:80 2048
```

你可以打开浏览器访问 http://127.0.0.1:8080 就能看到这个小游戏了。

5. 为了镜像可以到处分发（让全世界的人都可以使用），你可以将镜像上传至一个容器的镜像仓库，主流的镜像仓库是 [DockerHub](https://hub.docker.com/)。

登录镜像仓库：
```
docker login --username <你的账号>
```

将镜像改成对应账号的格式：
```
docker tag 2048 wonderflow/2048:v1
```

推送镜像：
```
docker push wonderflow/2048:v1
```

现在你也可以使用我推送好的镜像了：

```
docker run -p 8080:80 wonderflow/2048:v1
```

镜像制作好了，开始制作 Helm 包。

### 制作 Helm 包

1. 使用 helm 工具生成配置的脚手架：

```
helm create my-game-2048
```

此时会生成一个新的文件夹 “my-game-2048”，里面的文件都是 helm 自动生成的：

```
.
├── CONTRIBUTING.md
├── Dockerfile
├── README.md
...
├── my-game-2048
│   ├── Chart.yaml
│   ├── charts
│   ├── templates
│   │   ├── NOTES.txt
│   │   ├── _helpers.tpl
│   │   ├── deployment.yaml
│   │   ├── hpa.yaml
│   │   ├── ingress.yaml
│   │   ├── service.yaml
│   │   ├── serviceaccount.yaml
│   │   └── tests
│   │       └── test-connection.yaml
│   └── values.yaml
└── style
  ...
```

2. 修改镜像

在我们的场景下，默认生成的配置基本都是满足需求的，使用起来非常简单，就是把镜像改成我们的游戏镜像即可。

修改 `values.yaml` 文件里面的镜像，将其中 repository 一行的 nginx 改成 `wonderflow/2048`，tag 一行加上 `v1` 即可：

```
image:
- repository: nginx
+ repository: wonderflow/2048
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
- tag: ""
+ tag: "v1"
```

### 测试 Helm 包

1. 安装集群

Helm 包的运行依赖 Kubernetes 集群，一个最简单的安装方法是使用 `velad`.

[下载 velad](https://github.com/kubevela/velad)：

```
curl -fsSl https://static.kubevela.net/script/install-velad.sh
```

2. 部署集群

```
velad install --name foo --cluster-only
export KUBECONFIG=$(velad kubeconfig --name foo --host)
```

3. 测试 Helm 包 

本地文件夹部署：
```
helm install my-1024 my-game-2048
```

验证：
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=my-game-2048,app.kubernetes.io/instance=my-1024" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```

浏览器访问 http://127.0.0.1:8080 就可以看到这个游戏。

验证完毕，我们的 helm chart 就制作完成了，更复杂的制作可以参考 [Helm 官方文档](https://helm.sh/zh/docs/chart_template_guide/getting_started/)。

## 基于 GitHub 搭建 Helm 仓库做分发

为了让我们的 Helm Chart 可以被其他人使用，我们可以基于 [GitHub Pages](https://pages.github.com/) 功能，搭建一个 Helm 仓库。

1. 创建文件夹 Helm Chart 仓库文件夹

跟源代码区分开来，我们创建一个新的文件夹。

```
mkdir docs
```

2. 生成压缩包到指定目录
```
helm package my-game-2048 -d docs/
```

3. 生成访问的 index 文件
```
cd docs/
helm repo index .
```

4. 查看生成的内容：

```
$ cd ../
$ tree docs/
docs/
├── index.yaml
└── my-game-2048-0.1.0.tgz

0 directories, 2 files
```

4. 推送到你的自己的 Github 仓库（可以直接 fork 2048 游戏的仓库）：

```
git add .
git commit -m "my first helm repo"
git push 
```

经过我们的实验，结果仓库为： https://github.com/wonderflow/2048

5. 配置开启 GitHub Pages

配置路径： settings => Pages
设置 Sources 指向 master 分支，路径读取 `docs/` 。

![](/static/20220614/github-pages.jpg)


6. 至此我们就已经有了一个 helm 仓库了：

```
helm repo add my-repo https://<你的github账号>.github.io/2048
helm repo update
```

## 如何部署 Helm 包？

对于我本次的实验仓库而言： https://github.com/wonderflow/2048 ，已经可以直接使用这个仓库部署：

1. 添加仓库

```
helm repo add my-repo https://wonderflow.github.io/2048
helm repo update
```

2. 部署 Helm 应用

```
helm install my-2048 my-repo/my-game-2048
```

到这里，就完成了一个 Helm 的基本部署。