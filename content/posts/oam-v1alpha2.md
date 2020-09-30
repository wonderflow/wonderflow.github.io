---
title: "OAM v1alpha2 新版：平衡标准与可扩展性"
date: 2020-03-30T21:54:33+08:00
Tags: ["oam"]
---

OAM Spec 经历了近 3 个月的迭代，[v1alpha2](https://github.com/oam-dev/spec/releases/tag/v1.0.0-alpha.2) 版本终于发布啦。新版本在坚持 OAM Spec 平台无关的基础上，整体变得更 Kubernetes 友好化，很大程度上平衡了标准与可扩展性，更好的支持 CRD。如果你已经编写了现成的 CRD Operator，可以平滑的接入到 OAM 体系中，并且享受到 OAM 模型的红利。

<!-- more -->
目前 OAM 已经成为了包括阿里、微软、Upbond、谐云等多家公司构建云产品的核心架构。他们通过 OAM 构建了“以应用为中心”、用户友好化的 Kubernetes PaaS；充分发挥 OAM 的标准化与可扩展性，实现 OAM 核心Controller的同时，快速接入了已有的 Operator 能力；通过 OAM 横向打通多个模块，破除了原有 Operator 彼此孤立、无法复用的窘境。


- 了解 OAM 的背景及由来，可以参考文章[《深度解读 - 阿里巴巴基于 K8s 与 OAM 推进统一应用管理架构升级的教训与实践》](https://mp.weixin.qq.com/s/rRaHl5a5PU9Xg5psMservA?from=timeline&isappinstalled=0&scene=2&clicktime=1585550092&enterid=1585550092)
- OAM能为终端用户带来哪些价值可以参考文章 [《OAM 深入解读：OAM 为云原生应用带来哪些价值？》](https://mp.weixin.qq.com/s/7nkUqDNXzehIUnbGBPG8dQ)



下面言归正传，让我们来看一下 v1alpha2 到底做了哪些改动？


# 主要改动说明


为了方便大家阅读，这里只罗列了最主要的改动点，一些细节还是以上游 [OAM Spec Github](https://github.com/oam-dev/spec) 仓库为准。


## 术语说明


- CRD（Custom Resource Definition），在 OAM 中说的 CRD 是一种泛指的自定义资源描述定义。在 K8s 的 OAM 实现中可以完全对应 K8s 的 CRD，在非 K8s 的实现中，OAM 的 CRD 需要包含 APIVersion/Kind 并且能够描述字段进行校验。
- CR （Custom Resource），OAM 中的 CR 是 CRD 的一个实例，是符合 CRD 中字段格式定义的一个资源描述。在 K8s 的 OAM 实现中可以完全对应 K8s 的 CR，在 非 K8s 的实现中，可以需要对齐 APIVersion/Kind  和字段格式定义。



## 主要改动1 使用 Reference 模型定义 Workload、Trait 和 Scope


 v1alpha1 原先的方式是这样的：


```yaml
// 老版本，仅对比使用
apiVersion: core.oam.dev/v1alpha1
kind: WorkloadType
metadata:
  name: OpenFaaS
  annotations:
    version: v1.0.0
    description: "OpenFaaS a Workload which can serve workload running as functions"
spec:
  group: openfaas.com
  version: v1alpha2
  names:
    kind: Function
    singular: function
    plural: functions
  workloadSettings: |
    {
      "$schema": "http://json-schema.org/draft-07/schema#",
      "type": "object",
      "required": [
        "name", "image"
      ],
      "properties": {
        "name": {
          "type": "string",
          "description": "the name to the function"
        },
        "image": {
          "type": "string",
          "description": "the docker image of the function"
        }
      }
    }
```


在原先的模式中，group/version/kind 分别是字段，spec 的校验通过 jsonschema 表示，整体的格式实际上类似 CRD，但不完全一致。



---

新版 v1alpha2 中彻底改为了引用模型，通过 `WorkloadDefinition`  `TraitDefinition`  `ScopeDefinition` 的形式，描述了一个引用关系。可以直接引用一个 CRD，name 就是 CRD 的名称。对于非 K8s 的 OAM 实现来说，这里的名字则是一个索引，可以找到类似 CRD 的校验文件，校验文件中包含 apiVersion 和 kind，以及相应的 schema 校验。


- Workload
```yaml
apiVersion: core.oam.dev/v1alpha2
kind: WorkloadDefinition
metadata:
  name: containerisedworkload.core.oam.dev
spec:
  definitionRef:
    # Name of CRD. 
    name: containerisedworkload.core.oam.dev
```


- Trait
```yaml
apiVersion: core.oam.dev/v1alpha2
kind: TraitDefinition
metadata:
  name: manualscalertrait.core.oam.dev
spec:
  appliesToWorkloads:
    - containerizedworkload.core.oam.dev
  definitionRef:
    name: manualscalertrait.core.oam.dev
```


- Scope
```yaml
apiVersion: core.oam.dev/v1alpha2
kind: ScopeDefinition
metadata:
  name: networkscope.core.oam.dev
spec:
  allowComponentOverlap: true
  definitionRef:
    name: networkscope.core.oam.dev
```


注意：

1.  这里对于 K8s 的 OAM 实现来说，name 就是 K8s 里面 CRD 的name，由 `<plural-kind>.<group>` 组成。社区的最佳实践是一个 CRD 只有一个 version 在集群中运行，一般新 version 都会向前兼容，升级时都一次性替换到最新version。如果确实有 2 个 version 同时存在的场景，用户也可以通过 `kubectl get crd <name>` 的方式进一步选择。
1. Definition 这一层不面向 end user，主要给平台实现使用，对于非 K8s实现来说，如果存在多个 version 的场景，OAM 的实现平台可以给终端用户展示不同 version 的选择。



## 主要改动2 直接嵌入 K8s CR 作为 Component 和 Trait 实例


原先的方式在 Workload 和 Trait 层面我们都只把 CR 的 spec 部分拿出来，分别放在 `workloadSettings` 和 `properties` 字段里。


这样的方式虽然已经可以“推导”出 K8s CR，但是不利于 K8s 生态内的 CRD 接入，需要换种格式重新定义一遍 spec。


```yaml
// 老版本，仅对比使用
apiVersion: core.oam.dev/v1alpha1
kind: ComponentSchematic
metadata:
  name: rediscluster
spec:
  workloadType: cache.crossplane.io/v1alpha1.RedisCluster
  workloadSettings:
    engineVersion: 1.0
    region: cn
```


```yaml
// 老版本，仅对比使用
apiVersion: core.oam.dev/v1alpha1
kind: ApplicationConfiguration
metadata:
  name: custom-single-app
  annotations:
    version: v1.0.0
    description: "Customized version of single-app"
spec:
  variables:
  components:
    - componentName: frontend
      instanceName: web-front-end
      parameterValues:
      traits:
        - name: manual-scaler
          properties:
            replicaCount: 5
```



---



现在的方式则直接嵌入 CR，可以看到，在 `workload` 和 `trait` 字段下面是完整的 CR 描述。


```yaml
apiVersion: core.oam.dev/v1alpha2
kind: Component
metadata:
  name: example-server
spec:
  prameters:
    - name: xxx
      fieldPaths: 
        - "spec.osType"
  workload:
    apiVersion: core.oam.dev/v1alpha2
    kind: Server
    spec:
      osType: linux
      containers:
      - name: my-cool-server
        image:
          name: example/very-cool-server:1.0.0
        ports:
        - name: http
          value: 8080
        env:
        - name: CACHE_SECRET
```


```yaml
apiVersion: core.oam.dev/v1alpha2
kind: ApplicationConfiguration
metadata:
  name: cool-example
spec:
  components:
  - componentName: example-server
    traits:
    - trait:
        apiVersion: core.oam.dev/v1alpha2
        kind: ManualScalerTrait
        spec:
          replicaCount: 3
```


这样的好处很明显：


1. 可以很容易的对接现有 K8s 体系里的 CRD，甚至包括 K8s 原生的 `Deployment` （作为自定义 workload 接入）等资源。
1. K8s CR 层面的字段定义是成熟的，解析和校验也完全交由 CRD 体系。



这里大家注意到 traits 下面是 `[]trait{CR}` 而不是 `[]CR`  的结构，多了一层看似无用的 `trait` 字段，主要由如下2个原因：


1. 为后续在trait这个维度做扩展留下空间，比如可能的编排（ `ordering` ） 等。
1. 非K8s体系在这一层可以不严格按照CR的写法来，完全自定义，不绑定K8s描述格式。



## 主要改动3 参数传递使用 jsonPath 替换原先的 fromParam


**研发能够留出字段给运维覆盖**，一直是OAM很重要的功能。体现在 OAM Spec 的流程上就是研发在 Component 里面定义 parameter，运维在 AppConfig 里面通过 parameterValue 去覆盖对应的参数。


最初的参数传递，是在每个字段后面有个 `fromParam`  字段，对于支持了自定义schema后，这样的方式显然是无法覆盖所有场景的：


```yaml
// 老版本，仅对比使用
apiVersion: core.oam.dev/v1alpha1
kind: ComponentSchematic
metadata:
  name: rediscluster
spec:
  workloadType: cache.crossplane.io/v1alpha1.RedisCluster
  parameters:
  - name: engineVersion
    type: string
  workloadSettings:
    - name: engineVersion
      type: string
      fromParam: engineVersion
```
 
后来我们曾经提议过这样的方案：


```yaml
// 老版本，仅对比使用
apiVersion: core.oam.dev/v1alpha1
kind: ComponentSchematic
metadata:
  name: rediscluster
spec:
  workloadType: cache.crossplane.io/v1alpha1.RedisCluster
  parameters:
  - name: engineVersion
    type: string
  workloadSettings:
    engineVersion: "[fromParam(engineVersion)]"
```


这个方案最大的问题是 静态的 IaD (Infrastructure as Data) 里面加入了动态的函数，给理解和使用带来了复杂性。

---

经过多方面的讨论，在新方案里我们通过 JsonPath 的形式描述要注入的参数位置，在用户理解上保证了AppConfig 是静态的。


```yaml
apiVersion: core.oam.dev/v1alpha2
kind: Component
metadata:
  name: example-server
spec:
  workload:
    apiVersion: core.oam.dev/v1alpha2
    kind: Server
    spec:
      containers:
      - name: my-cool-server
        image:
          name: example/very-cool-server:1.0.0
        ports:
        - name: http
          value: 8080
        env:
        - name: CACHE_SECRET
          value: cache
  parameters:
  - name: instanceName
    required: true
    fieldPaths:
    - ".metadata.name"
  - name: cacheSecret
    required: true
    fieldPaths:
    - ".workload.spec.containers[0].env[0].value"
```


fieldPaths 是个数组，每个元素定义了参数和对应 Workload 里的字段。


```yaml
apiVersion: core.oam.dev/v1alpha2
kind: ApplicationConfiguration
metadata:
  name: my-app-deployment
spec:
  components:
    - componentName: example-server
      parameterValues:
        - name: cacheSecret
          value: new-cache
```


在 AppConfig 中还是走 parameterValues 去覆盖 Component 中的 parameter。




## 主要改动4 ComponentSchematic 名称改为 Component


原先组件这个概念叫 ComponentSchematic，这样命名的主要原因是里面混了一些语法描述和选择，比如针对Core Workload（ `container`）和针对扩展 Workload （ `workloadSettings` ），写法上不一样的，`container`里是定义具体的参数，`workloadSettings`更像是 schema（参数怎么填的说明）。 v1alpha1 版本的 WorkloadSetting 还融入了 type/description之类的，更显得模棱两可。
```yaml
// 老版本，仅对比使用
apiVersion: core.oam.dev/v1alpha1
kind: ComponentSchematic
metadata:
  name: rediscluster
spec:
  containers:
     ...
  workloadSettings:
    - name: engineVersion
      type: string
      description: engine version
      fromParam: engineVersion
     ...
```



---

在 v1alpha2 版本中，组件这个概念改为 Component，明确为 Workload 的实例，所有语法定义都是在WorkloadDefinition 中引用的实际 CRD 定义的。


在 K8s 实现中，WorkloadDefinition 就是引用 CRD ，Component.spec.workload 里就是写 CRD 对应的实例 CR。
```yaml
apiVersion: core.oam.dev/v1alpha2
kind: Component
metadata:
  name: example-server
spec:
  workload:
    apiVersion: core.oam.dev/v1alpha2
    kind: Server
    spec:
   ...
```


## 主要改动5 Scope 由 CR 单独创建，不再由 AppConfig 创建


v1alpha1 中的 Scope 是由 AppConfig 创建的，从例子中可以看出，本质上它也是个 CR，可以“推导”创建出 CR来。但是由于 Scope 的定位是可以容纳不同 AppConfig 中的 Component，且Scope 本身并非一个App，所以使用 AppConfig 创建 Scope 一直是不太合适的。


```yaml
// 老版本，仅对比使用
apiVersion: core.oam.dev/v1alpha1
kind: ApplicationConfiguration
metadata:
  name: my-vpc-network
spec:
  variables:
    - name: networkName
      value: "my-vpc"
  scopes:
    - name: network
      type: core.oam.dev/v1alpha1.Network
      properties:
        network-id: "[fromVariable(networkName)]"
        subnet-ids: "my-subnet1, my-subnet2"
```



---

v1alpha2 新版本全面使用 CR 来对应实例，为了让Scope的概念更清晰，更方便对应不同类型的Scope，将 Scope 拿出来直接由ScopeDefinition 定义的CRD 对应的 CR 创建。例子如下所示：


```yaml
apiVersion: core.oam.dev/v1alpha2
kind: ScopeDefinition
metadata:
  name: networkscope.core.oam.dev
spec:
  allowComponentOverlap: true
  definitionRef:
    name: networkscope.core.oam.dev
```


```yaml
apiVersion: core.oam.dev/v1alpha2
kind: NetworkScope
metadata:
  name: example-vpc-network
  labels:
    region: us-west
    environment: production
spec:
  networkId: cool-vpc-network
  subnetIds:
  - cool-subnetwork
  - cooler-subnetwork
  - coolest-subnetwork
  internetGatewayType: nat
```


在AppConfig中使用 scope 引用如下所示:


```yaml
apiVersion: core.oam.dev/v1alpha2
kind: ApplicationConfiguration
metadata:
  name: custom-single-app
  annotations:
    version: v1.0.0
    description: "Customized version of single-app"
spec:
  components:
    - componentName: frontend
      scopes:
        - scopeRef:
            apiVersion: core.oam.dev/v1alpha2
            kind: NetworkScope
            name: my-vpc-network
    - componentName: backend
      scopes:
        - scopeRef:
            apiVersion: core.oam.dev/v1alpha2
            kind: NetworkScope
            name: my-vpc-network
```




## 主要改动6 移除 Variable 列表和 [fromVariable()] 动态函数


v1alpha1 版本中有 Variable 是为了 AppConfig 里开源引用一些公共变量减少冗余，所以加入了 Variable 列表。但实践来看，减少的冗余并没有明显减少 OAM spec 的复杂性，相反，增加动态的函数显著增加了复杂性。


另一方面，fromVariable 这样的能力完全可以通过 helm template/ kustomiz 等工具来做，由这些工具渲染出完整的 OAM spec，再进行使用。


所以这里 variables 列表和相关的 fromVariable 均去掉，并不影响任何功能。


```yaml
// 老版本，仅对比使用
apiVersion: core.oam.dev/v1alpha1
kind: ApplicationConfiguration
metadata:
  name: my-app-deployment
spec:
  variables:
    - name: VAR_NAME
      value: SUPPLIED_VALUE
  components:
    - componentName: my-web-app-component
      instanceName: my-app-frontent
      parameterValues:
        - name: ANOTHER_PARAMETER
          value: "[fromVariable(VAR_NAME)]"
      traits:
        - name: ingress
          properties:
            DATA: "[fromVariable(VAR_NAME)]"
```




## 主要改动7 用 ContainerizedWorkload 替代原来的六种核心Workload


因为现在已经统一用 WorkloadDefinition 定义 Workload，Component 变成了实例，所以原先的六种核心Workload 实际上都变成了同一个 WorkloadDefinition，字段描述完全一样，唯一的不同是对 trait 约束和诉求不一样。所以原先的六种核心 Workload 的 spec，统一修改为一种名为 ContainerizedWorkload 的 Workload 类型。

同时，这里计划要通过增加 policy 这样的概念，来让研发表达对运维策略的诉求，即 Component 中可以表达希望增加哪些 trait。

```yaml
apiVersion: core.oam.dev/v1alpha2
kind: WorkloadDefinition
metadata:
  name: containerizedworkloads.core.oam.dev
spec:
  definitionRef:
    name: containerizedworkloads.core.oam.dev
```


一个使用 ContainerizedWorkload 示例：


```yaml
apiVersion: core.oam.dev/v1alpha2
kind: Component
metadata:
  name: frontend
  annotations:
    version: v1.0.0
    description: "A simple webserver"
spec:
  workload:
    apiVersion: core.oam.dev/v1alpha2
    kind: ContainerizedWorkload
    metadata:
      name: sample-workload
    spec:
      osType: linux
      containers:
      - name: web
        image: example/charybdis-single:latest@@sha256:verytrustworthyhash
        resources:
          cpu:
            required: 1.0
          memory:
            required: 100MB
        env:
        - name: MESSAGE
          value: default
  parameters:
  - name: message
    description: The message to display in the web app.  
    required: true
    type: string
    fieldPaths:
    - ".spec.containers[0].env[0].value"
```




# 下一步计划


1. 应用级别的组件间参数传递和依赖关系（workflow）[https://github.com/oam-dev/spec/pull/326](https://github.com/oam-dev/spec/pull/326)
1. Policy方案，便于研发在 Component 对 trait 提出诉求。[https://docs.google.com/document/d/1d3kVBFyf7yf-YzmVqkL1hr_qcIVXf6BUJLWKu9NJwvE/edit#heading=h.hh69i79bk4p3](https://docs.google.com/document/d/1d3kVBFyf7yf-YzmVqkL1hr_qcIVXf6BUJLWKu9NJwvE/edit#heading=h.hh69i79bk4p3)
1. Component 增加版本的概念，同时给出 OAM 解决应用版本发布相关方式。 [https://github.com/oam-dev/spec/issues/336](https://github.com/oam-dev/spec/issues/336) [https://github.com/oam-dev/spec/issues/341](https://github.com/oam-dev/spec/issues/341)



# 常见 FAQ


1. 我们原有平台改造为 OAM 模型实现需要做什么？



对于原先是 K8s 上的应用管理平台，接入改造为 OAM 实现可以分为两个阶段：
1）实现 OAM ApplicationConfiguration Controller（简称AppConfig Controller），这个Controller 同时包含 OAM 的 Component、WorkloadDefinition、TraitDefinition、ScopeDefinition 等 CRD。AppConfig Controller 根据 OAM AppConfig中的描述，拉起原有平台的 CRD Operator.
2) 逐渐将 原先的 CRD Operator 根据关注点分离的思想，分为 Workload 和 Trait。同时接入和复用 OAM 社区中更多的Workload、Trait，丰富更多场景下的功能。
 


2. 现有的 CRD Operator 为接入 OAM 需要做什么改变？

**
**    现有的 CRD Operator** 功能上可以平滑接入OAM体系，比如作为一个独立扩展 Workload 接入。但是为了更好的让终端用户体会到 OAM 关注点分离的好处，我们强烈建议 CRD Operator 根据研发和运维不同的关注点分离为不同的 CRD，研发关注的 CRD 作为 Workload 接入 OAM，运维关注的 CRD 作为 Trait 接入 OAM。




# 作者简介


孙健波 阿里云技术专家，是 OAM 规范的主要制定者之一，致力于推动云原生应用标准化。同时也在阿里巴巴参与大规模云原生应用交付与应用管理相关工作。团队诚邀应用交付、Serverless、PaaS 领域的专家加入，欢迎联系  jianbo.sjb AT alibaba-inc.com




