---
title: "CUE 基础入门"
date: 2020-12-15T14:21:16+08:00
---

磨刀不误砍柴工，在正式开始介绍 CUE 之前，可以先做下基本的准备工作。

* 安装：首先确保你已经安装了 cuelang： https://cuelang.org/docs/install/
* 自动格式化：如果你使用 Goland 或者类似的 JetBrains IDE，还可以额外配置自动格式化的方式：https://wonderflow.info/posts/2020-11-02-goland-cuelang-format/


## 基本操作

先看一下cuelang的基本数据样子。
```
a: 1.5
b: 1
d: [1, 2, 3]
g: {
	h: "abc"
}
```

我们把上面的内容写入到文件并命名为 "first.cue".

* 格式化 cuelang 的文件

```
cue fmt first.cue
```

* 校验写法是否正确

```
cue vet first.cue
```

* 计算输出结果

```
cue eval first.cue
```

* 计算输出单个变量的结果

```
cue eval -e a first.cue
```

* 数据导出（默认为json）

```
cue export first.cue
```

* 指定 yaml 格式导出
```
cue export first.cue --out yaml
```

* 导出某个变量
```
cue export first.cue -e a --out yaml
```


## 类型 与  “集合” 式编程

在熟悉了 cuelang 基本操作以后，我们来看下它的常用类型和写法。

你可以直接定义一个“schema”，如：
```
a: float
```

也可以直接写“data”：

```
// 浮点型
a: 1.5
```

可以看到 cuelang 里面“数据data”和“定义schema”的写法是一样的。

```
a: float
a: 1.5
```

把两个文件写到一起，然后计算下最后的结果：

```
$ cue eval first.cue 
a: 1.5
```

可以看到变量 `a` 的结果就确定下来是 1.5 。

这个过程的本质是两个集合做了一个“相交”的操作。 第一个集合 `a: float` 代表了 a 是所有浮点数。 第二个集合 `a: 1.5` 则把 a 这个变量的值缩小到只剩下一个确定的值 `1.5`. 所以最后的结果是 `1.5`.

我们可以改成别的类型试试:

```
a: string
a: float
```

再运行一下计算：

```
$ cue eval first.cue
a: conflicting values string and 1.5 (mismatched types string and float):
    ./first.cue:1:4
    ./first.cue:2:4
```

所以cuelang里面的计算实际上就是不断在做集合相交，把结果集不断缩小。如果最后两个集合相交没有任何结果，用 `_|_`表示（空集合）。

除了上述的几个例子以外，cuelang 的常用数据类型还包括：

```
// 浮点型
a: 1.5

// 整型
b: 1

// 字符串类型
c: "blahblahblah"

// 数组
d: [1, 2, 3, 1, 2, 3, 1, 2, 3]

// Bool 型
e: true

// 结构体类型
f: {
	a: 1.5
	b: 1
	d: [1, 2, 3, 1, 2, 3, 1, 2, 3]
	g: {
		h: "abc"
	}
}

// null
j: null
```

也可以用`#`符号明确指定一个类型：

```
#a: string
a: 1.2
```

这里指定的 `#a` 是明确为一种类型，计算时并不会输出结果，与变量 `a` 也没有任何关系:

```
$ cue export first.cue
{
    "a": 1.2
}
```


## 操作符

### “或”操作

```
a: 1 | 2
```
表示 a 的值既可以是 1，也可以是 2.

```
a: string | int
```
表示 a 的值是 string 或者 int 类型。

### “默认值”

```
a: *1 | int
```

把 `*` 放在具体的数值前面表示默认值，通常情况默认值要和“或操作” 一起使用，表示 a 是 int 类型，默认值是 1.

### “可缺省”参数

还有一种特殊的情况是默认可以不存在（即“可缺省”）：

```
a ?: int

my: {
  x ?: string
  y : int
  z ?:float
}
```

### “与”操作

```
a: *1 | int
b: 3
c: a & b
```

将 a 和 b 两个集合相交就是 用 `&` 操作符。

```
$ cue eval first.cue
a: 1
b: 3
c: 3
```

### 条件判断

```
price: number
feel: *"good" | string
// Feel bad if price is too high
if price > 100 {
    feel: "bad"
}
price: 200
```

```
$ cue eval first.cue
price: 200
feel:  "bad"
```

### 循环

```
#a: {
	"hello": "Barcelona"
	"nihao": "Shanghai"
}

for k, v in #a {
	"\(k)": {
		nameLen: len(v)
		value:   v
	}
}
```

除了循环的用法外，这里用 双引号 和 括号 （`"\(k)"`）来做了个内部的计算和字符串转换，还用到了一个计算字符串长度的cuelang内置函数 `len`。这些内置的内容不用深究，可以后续深入了解时再学习。

```
$ cue export first.cue
{
    "hello": {
        "nameLen": 9,
        "value": "Barcelona"
    },
    "nihao": {
        "nameLen": 8,
        "value": "Shanghai"
    }
}
```

## 变量引用与模板

至此，我们对cuelang的类型和各种操作基本有了完整的认识，我们来试着使用上面学到的内容，试着构建一个实际的模板。

1. 定义一个结构体类型，包含2个字符串成员。
```
#Config: {
	name:  string
	value: string
}
```

2. 申明一个 context 变量，其中一个是字符串成员，另外一个是数组，数组中的元素则使用在第一步中定义的结构体类型`#Config`。
```
parameter: {
	name: string
	image: string
	config: [...#Config]
}
```

3. 定义一个模板，使用上面的 context 变量。
```
template: {
	apiVersion: "apps/v1"
	kind:       "Deployment"
	spec: {
		selector: matchLabels: {
			"app.oam.dev/component": parameter.name
		}
		template: {
			metadata: labels: {
				"app.oam.dev/component": parameter.name
			}
			spec: {
				containers: [{
					name:  parameter.name
					image: parameter.image
					if context["config"] != _|_ {
						env: parameter.config
					}
				}]
			}}}
}
```

4. 熟悉 K8s 的朋友可能已经发现，上面模板有点像 K8s 的Deployment。实际上这个就是一个 K8s Deployment 的模板。而模板中可替换的部分就是在 parameter 中的内容。让我们来为 parameter 赋值。
```
parameter:{
   name: "mytest"
   image: "nginx:v1"
   config:[{name:"a",value:"b"}]
}
```

5. **让我们依次把1~4步中的内容粘贴到一个cue文件中去，然后执行**：

```
$ cue export first.cue -e template --out yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: mytest
        env:
        - name: a
          value: b
        image: nginx:v1
    metadata:
      labels:
        app.oam.dev/component: mytest
  selector:
    matchLabels:
      app.oam.dev/component: mytest
```

一个完整的 K8s deployment yaml 已经出现。

至此，我们已经学会了一些 cuelang 的基本用法并试着编写了一个Deployment模板。
