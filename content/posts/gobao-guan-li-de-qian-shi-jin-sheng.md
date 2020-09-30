+++
title = "Go包管理的前世今生"
date = "2017-09-07T10:19:43Z"
slug = "2017/09/07/gobao-guan-li-de-qian-shi-jin-sheng"
Categories = []
+++

原文首发自InfoQ，[《Go包管理的前世今生》](http://www.infoq.com/cn/articles/history-go-package-management)


说实话，Golang对一个新人真的挺不友善的，因为一上手要了解的概念。你看人家Java，上来一个项目`mvn install`一下就完事了，赶紧利落。但是Golang就麻烦了，你得先了解什么是GOPATH。我当年刚接触Golang真正开始做项目的时候，只知道按要求配置环境变量，对GOPATH真正理解可能都是好几个月以后的事情了。说白了，还是因为懒。真正做项目的人，有多少有耐心砍柴磨刀，出现一个东西就研究半天啊，我们只是想要Copy-Paste而已。

但是不得不承认，对于今天讨论的Go包管理的话题，如果你想理解Golang的包管理机制，连GOPATH都不想充分理解一下，那可能真的不需要看这篇文章，下次遇到了照着README一步步老老实实来就行了。照着README文档搞不定怎么办？给项目维护者提BUG啊！

## GOPATH

言归正传，还是回到GOPATH的理解上来。那么，GOPATH是什么？有什么用？本质上GOPATH是一个系统的环境变量，就是Go语言用来存放代码依赖的地方。

很多人搞不清 GOPATH 、 GOROOT 的区别，其实没必要理解的很复杂。当Go语言的安装包刚下载完毕的时候，你把它解压或者直接安装到的那个目录，就是 GOROOT 目录，此时你需要做一些额外的配置，将GOROOT这个环境变量设置一下。当然，对应的bin目录你也得设置一下，否则操作系统找不到`go`的执行文件。

比如你解压后安装到 `/opt/go/` 目录下了，又或者在Windows下面你安装到了C盘 `C:\\GO`，都是OK的，区别只是不同操作系统环境下设置的方法不同而已。具体怎么设置我就不赘述了。

到此为止，你已经可以忘记 `GOROOT` 这个事情了，因为已经解决了所有跟它有关的事情。但是我们还是要解释下，为什么要设置这个东西？道理很简单，Golang的很多默认机制都很喜欢从环境变量里面去读内容，设置了 `GOROOT` 环境变量，相当于告诉所有读取这个变量的程序我们Golang的源码位置，便于代码的引用。可以理解为 GOROOT 就是三原色，用它可以组合出很多不同的色彩，是最初始的代码依赖。GOROOT 里面的很多代码都是系统驱动程序以及系统调用。

那么我们有了三原色，想要配出更多的颜色，我们调配颜色的过程中组合出的颜色，也就是新写出来的代码包放在哪里呢？ 你一定已经猜到了，这就是 `GOPATH` 目录的作用。所有Golang安装包以外的代码，无论是你自己写的，还是第三方的无比成熟的包，都需要放置在`GOPATH`下面。

所以你不要再问为什么我直接 `git clone` 下来的代码怎么各种报错说找不到依赖啦，你设置 `GOPATH` 了吗？

那么怎么设置 `GOPATH` 呢？ 这就根据个人口味定了，很随意。有的人选择用隐藏目录，比如 `~/.go/` 作为`GOPATH`，也有的人设置在`GOROOT`的隔壁，新建了个目录叫`/opt/gopath`，对于Windows用户来说也是如此，当然你完全可以设置环境变量把自己在自己的D盘下也创建一个叫gopath的目录`D:\\Gopath`，然后设置下环境变量。而我个人则更偏爱`/home/sunjianbo/gopath`这个目录，也就是`~/gopath`。

那么我可不可以有多个GOPATH目录呢？当然可以，设置系统环境变量的时候就是可以放多个值的嘛。

你完全可以设置 `export GOPATH=~/gopath1:~/gopath2:~/gopath3`。在配置的这些目录中，Go程序会依次去寻找有没有对应的依赖包。

所以是不是有的读者已经想明白最原始的包管理方法了呢？

公布答案，就是每个项目做一个 `GOPATH`。

具体而言，假设我们有个项目叫 `tastego`,我们在里面写个脚步，内容如下：
```
export GOPATH=$PWD/tastego:$GOPATH
```

只要一行，简单到任性。当然，最好再加一行，把GOPATH下的bin路径加上，这样`go install`出来的内容也能开箱即用。

```
export PATH=$PATH:$PWD/tastego/bin
```

对于每个项目的依赖，分门别类的放在对应的源码目录下。

```
tastego/
    src/
        github.com/
            wonderflow/...
        qiniu/...
        golang.org/...
    bin/
    pkg/
```

所以，你可能不信，最开始Go官方根本没有提供什么包管理机制的时候，好多Go语言玩家，都是用这样的方式在玩。照样玩的顺风顺水，数十上百万行代码不在话下。

但是值得一提的是，我们观察到 `GOPATH=$PWD/logkit:$GOPATH` 这个结构是一个递归的结构，不仅可以这么写，换个方式`GOPATH=$GOPATH:$PWD/logkit` 也是有效的。但是这两种写法的利弊是不同的。先说结论，这两种方式我都不推荐。

* `GOPATH=$PWD/logkit:$GOPATH` 这个写法是当前项目的目录在前，所以遇到新的项目，永远在gopath的最前面。这个很大的好处就是当你遇到多个包名称完全相同的时候，用的默认是最前面的包，所以包冲突的概率会降到最低。坏处也显而易见，你 `go get` 下来的代码包，会进入到最前面的这个目录，导致每次 `go get`的位置是不确定的，尤其是环境变量被无意中clean后，你甚至需要重新 `go get`一遍。
* `GOPATH=$GOPATH:$PWD/logkit` 这个写法就保证了顺序的稳定性，后来的包一定在后面，但是弊端就是你很快就要出现包版本冲突的问题了。

当然，上述做法还有一个巨大的缺点，就是当你的项目大了以后，你可能不得不把半个Github的代码都放在你的单个代码库（Repo）下面。

## 百花齐放

终于，在Golang官方面对GOPATH管理的各种乱象始终无动于衷的时候，社区看不下去了，相继出现各类包管理工具。

2013年的时候，大名鼎鼎的[Gedep](https://github.com/tools/godep)工具诞生了。这是社区第一个包管理工具，受到了大家的热烈追捧。原理非常简单，就是把我们上一节讲的方式，通过脚本来实现。把所有的源码保存到一个叫 `Godeps/_workspace/` 的目录下，然后将这个 `Godeps/_workspace/` 目录作为唯一的 `GOPATH`. 提供了一系列包括 `godep save` 、 `godep restore` 这样方便快捷的把所有依赖包都保存到 `Godeps/_workspace/` 目录下，同时又可以快速的通过`godep` 恢复 `GOPATH`，相当于对`GOPATH`及源码的版本做了一次快照。当然，它并没有解决你一次`git clone`需要下载半个github的难题。

但是就在这一年，Docker开始火起来了，所以好多Golang项目构建Dockerfile做测试的起手式就是：

```
ADD . /app/xxx/
ENV GOPATH /app/xxx/Godep/_workspace
```

一下子所有依赖全都有了，确实很方便。

但是在当时（2013~2014年），这个工具依然没有解决我的版本问题，依然没有解决我依赖的包如果还依赖许多别的包怎么办的问题。于是到了2014年，号称 `参照了其他语言的包管理工具**最佳实践**` 的 [glide](https://github.com/Masterminds/glide) 包管理工具诞生了。

那么它到底凭什么号称自己是`最佳实践`呢？它的`最佳`体现在它的包管理哲学上，一旦你安装完成`glide`以后，一个`glide create`命令，就会完全搞定所有包依赖的问题。至于说构建时有多个版本可以用，依赖怎么选、选哪个的问题，它会帮你搞定。

那如果我希望自己指定包版本呢？当然也没问题，只要在 `glide.yaml` 配置文件中填写好你对应的包版本、或者版本的范围，glide 会在这个范围内选定。

所以对于从零构建项目的新用户来说，`glide` 一定是个不错的选择，它帮你处理了所有的包依赖问题。但是对于一个已经有一些历史包袱的项目来说，使用 `glide` 可能就有些尴尬了。尤其是你的项目里本来就存在一些依赖冲突的问题，glide 还非要帮你确定使用哪个依赖，导致最终还是运行不起来。

所以[有人](https://github.com/Masterminds/glide/issues/249)就问了，“可不可以不要绝对的给我最佳实践，把我的依赖全导进来，一步步来行么？” 官方就说，"我们的设计哲学是glide管理下的每一步都是可执行的，不能只走半步！" 但是您没给人解决问题啊，最终还是执行不了，这就很僵……

也许正是理念的不同，所以 `glide` 号称“最佳实践”但却并未一统江湖。

这段时间还有诸如 `gopkg.in`、`gom` 等包管理工具诞生，其理念都比最初的`godep`先进，你说与 `glide` 比没有更多特色，都加入了包版本的概念。

## Vendor

时间走到了2015年，Golang官方终于看不下去了，在推出go1.5版本的同时，首次实验性质的加入了 `vendor机制` 功能。当然，这个功能毕竟是实验性质的，默认情况下是关闭的，导致大多数用户实际上根本不会用它。直到2016年，在官方推出go1.6版本的时候，`vendor机制` 才默认变成开启的状态。

那么到底什么是 `vendor机制` 呢？ 通俗的说，就是在你的项目中包含了一个`vendor`文件夹，go语言会把它默认为一个GOPATH。于是，你就可以在里面放你的依赖库啦。

举例来说，假设你托管在github上的项目是这样的，项目名称为tastego:

```
tastego/
    main.go
    common/
        common.go
    util/
        util.go
```

其中除了 main.go 主函数以外，还包含2个自己写的库(package)，一个是common，一个是util，那么为了让项目可以正常编译，这2个库应该在GOPATH中，那么实际上在GOPATH的结构下，你的项目目录是这样的（wonderflow是我的github ID）：

```
$GOPATH/
    src/
        github.com/
            wonderflow/
                tastego/
                    main.go
                    common/
                        common.go
                    util/
                        util.go
```

然后加入了vendor机制后，你的项目目录下增加一个vendor的文件夹，里面可以放别的依赖，形式上就是：

```
$GOPATH/
    src/
        github.com/
            wonderflow/
                tastego/
                    main.go
                    common/
                        common.go
                    util/
                        util.go
                    vendor/
                        github.com/
                        qiniu/
                            ...
                    golang.org/
...
```

让我们再回顾一下本文刚开始描述的基于GOPATH的包管理方法：

```
tastego/
    src/
        github.com/
            wonderflow/...
        qiniu/...
        golang.org/...
    bin/
    pkg/
```

看上去非常相似，只是有了vendor，就有了官方的正名！并且你再也不需要手工（半手工）修改GOPATH，项目的形态也跟以前的统一起来了，好处显而易见。

但是问题就真的解决了吗？实际上并没有全部解决问题，反而由于在随后2016、2017年，vendor机制成为正式的Go规则，问题日益严重。

1. 嵌套的vendor目录问题：vendor目录下面的项目里面的vendor目录怎么办？
2. vendor机制本身没有版本概念，不同版本间类型不兼容问题依旧存在。
3. 与其他 GOPATH 下的包init函数冲突问题：出现了相同的包，重复的init() 函数又怎么办？

所以Golang团队成员也召开了大会，非常赞同社区里各种包管理工具的理念，确实有必要对包管理提出一个统一的规则，来解决上面的问题。但是问题不是没有规则，而是规则太多了。往往就是一个意见不合，一下子就杀出来一个新的工具。仅官方推荐的包管理工具就有15种之多。

* [dep](https://github.com/golang/dep)
* [manul](https://github.com/kovetskiy/manul) - Vendor packages using git submodules.
* [Godep](https://github.com/tools/godep)
* [Govendor](https://github.com/kardianos/govendor)
* [godm](https://github.com/hectorj/godm)
* [vexp](https://github.com/kr/vexp)
* [gv](https://github.com/forestgiant/gv)
* [gvt](https://github.com/FiloSottile/gvt) - Recursively retrieve and vendor packages.
* [govend](https://github.com/govend/govend) - Manage dependencies like `go get` but for `/vendor`.
* [Glide](https://github.com/Masterminds/glide) - Manage packages like composer, npm, bundler, or other languages.
* [Vendetta](https://github.com/dpw/vendetta)
* [trash](https://github.com/rancher/trash)
* [gsv](https://github.com/toxeus/gsv)
* [gom](https://github.com/mattn/gom)
* [Rubigo](https://github.com/yaa110/rubigo) - Golang vendor utility and package manager

具体的可以参见官方的wiki页面： https://github.com/golang/go/wiki/PackageManagementTools

所以Go官方也开始尝试把包管理做成 go tools 工具链中的一个，官方的包管理工具就是 [dep](https://github.com/golang/dep) 但是目前这个项目还不成熟，还没有纳入到工具链中。

但是官方的建议已经很明显了，让大家尽量使用包管理工具去引入依赖，当然最好是尽量使用标准库；另一方面则是尽量使用现有的包管理工具，而不是自己再去造一套规则。

所以，我们也来学习一下包管理工具该怎么用，经过多次对比调研，笔者推荐
[Govendor](https://github.com/kardianos/govendor) 工具，所以也以之为例介绍。

## Govendor

Govendor 本质上就是把源码拷贝到vendor目录下，通过在vendor目录维护一个 vendor.json 的文件，指定使用的包版本。整个目录结构清晰，在同步到github时，既可以把代码直接全部包含到项目中，也可以用 `.gitignore` 忽略依赖的库并通过 `govendor sync` 同步。

### 安装

安装就跟所有golang的工具一样，`go get` 即可.

```
go get -u github.com/kardianos/govendor
```

### 初始化项目

对于一个现有的项目，没有使用过任何包管理工具的话，开始使用Govendor 非常简单。

进到项目目录下，执行初始化：

```
cd "my project in GOPATH"
govendor init
```

将现有依赖加入到当前项目的 vendor 中管理。

```
govendor add +external
```

此时，你已经顺利将现有项目切换到govendor管理了。


### 项目过程中的常用命令

初始化项目完毕后，就到了项目常规管理阶段，通常情况下，会有下列这些场景的需求。

#### 添加依赖

* 如果你本地GOPATH中已经存在，使用 `govendor add`

```
# 指定版本的commit，包名后跟 @ 符号加上 commit ID
govendor add golang.org/x/net/context@a4bbce9fcae005b22ae5443f6af064d80a6f5a55
# 指定版本名称，包名后跟 @ 符号加上版本名称或分支名称
govendor add golang.org/x/net/context@v1   # Get latest v1.*.* tag or branch.
```

* 如果你本地不存在，使用 `govendor fetch`， 其他指定版本的方式与 `govendor add` 相同

* 一次性把所有项目的依赖库全加载进来，就是我们初始化时介绍的命令。

```
govendor add +external
```

#### 移除依赖

* 移除一个依赖

```
govendor remove golang.org/x/net/context
```

* 移除所有项目已经不用的依赖

```
govendor remove +unused
```

#### 更新依赖

当然你可以选择 `govendor remove` ，然后再 `govendor add`。

你还可以直接使用 `govendor update` 更新本地GOPATH中已经更新的包。

若本地不是最新的或不存在，请用 `govendor fetch` 更新。

```
govendor update golang.org/x/net/context@a4bbce9fcae005b22ae5443f6af064d80a6f5a55
```

#### 同步govenodr包

一般情况下，开源项目的协作过程中，其他人更新了项目的govendor，那么你也要同步过来，直接使用下面的命令即可。

```
govendor sync
```

#### 查看本地依赖以及包状态

通常情况下拿到一个项目可能会想要直观的了解他有哪些依赖关系，使用 `govendor list`即可查看。

纯粹使用 `govendor list` 价值不大。

有意思的是，Govendor给Golang的依赖包加入了状态描述，结合各类vendor的状态参数进行各类操作就很有意思。

* 状态一共有如下几类：

```
	+local    (l) 仅存在项目中的包
	+external (e) 在GOPATH中有，但是项目中没有的包
	+vendor   (v) 在项目vendor目录中的包
	+std      (s) 使用golang标准库

	+excluded (x) 项目中不包含且明确申明要排除在外的包
	+unused   (u) 在vendor目录中，但实际项目没有用到的包
	+missing  (m) 项目中用到但是找不到的包（此时需要govendor fetch获取）

	+program  (p) 带有main函数入口的包

	+outside  所有外部包组合， 包括 （+external +missing）
	+all     列出所有的包
```

这些状态信息可以与其他命令连用，比如

```
govendor add +external
```

```
govendor remove +unused
```

最酷的是，状态还可以做逻辑 `与` 以及 `或` 的操作，比如：

```
+local,program (local AND program) 
表示项目中的包同时又是主函数入口

+local +vendor (local OR vendor) 
表示项目中的包以及vendor中的包

+vendor,program +std ((vendor AND program) OR std) 
表示vendor中的包同时带有主函数入口，再加上标准库的包

+vendor,^program (vendor AND NOT program)
表示vendor目录中的包，但是不包含有主函数入口的包
```

* 查看包之间的依赖关系

使用 `-v` 参数可以查看一个包被哪些包依赖：

```
govendor list -v
```

那么反过来，你可能想知道一个包依赖了哪些包？这个是go工具链里面提供的方法，直接使用 `go list`

比如：

`go list -f '{{ .Imports }}' github.com/wonderflow/tastego`

* 查看包的实际路径

通过 `-p` 参数可以看到包所在的实际文件路径，区别于import时填写的路径，实际路径可以快速找到你引用的包位置。

查看所有当前项目的包：

```
govendor list -p -no-status +local
```

至此，我们，Govendor的常用命令已经介绍完了，相信掌握了这些，在今后的项目中管理各种依赖包，你一定游刃有余。
