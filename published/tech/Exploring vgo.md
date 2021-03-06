已发布：https://studygolang.com/articles/12637

# 探索 vgo

昨天，Russ Cox 发布了 [vgo](https://research.swtch.com/vgo)，作为一个现有 go 构建命令的继任者，添加了一直缺失的包版本管理功能。虽然它只是一个大胆的尝试，但是在大家都认为 [dep](https://github.com/golang/dep) 将要成为 Go 语言官方正式的包管理工具的时候，它的出现多少让大家有点意外。Russ 写的 [vgo 简介](https://research.swtch.com/vgo-intro) 和一起发布的 [vgo 使用指南](https://research.swtch.com/vgo-tour) 是了解 vgo 很好的参考资料，尽管许多人对文章中的一些观点有些误解，我还是强烈建议第一次接触 vgo 的朋友读一下。（译注：vgo 的系列文章，Go 中文网专栏有中译文，地址：https://studygolang.com/subject/52）

第一次读那篇文章的时候，我和大家一样，对文章里提到的一些观点比较困惑，总是觉得哪里有些不对，但是又不能很明确的表达出来。为了更加深入地理解 vgo 是什么，我决定在读 [vgo 使用指南](https://research.swtch.com/vgo-tour) 的同时，下载 vgo 的源码，做一些代码测试。大家可以从 Github 上下载到我在这期间写的一些 [代码](https://github.com/joncalhoun?tab=repositories)

当我对 vgo 有了更好的理解后，我发现大家提出的一些假设都是不对的，这些在 [vgo 使用指南](https://research.swtch.com/vgo-tour) 中都有写到，而且只要通过少量的实践，就更不会有困惑了。但是我相信在消除这些疑虑之前，大家是不会去使用 vgo 的。这就是我写这篇文章的目的，来阐明一些我看到的大家提出的一些误解。同时也顺便表达一下我对 vgo 这个项目由衷的激动之情

Note：我相信 Russ 正在写一些文章来对 vgo 中的一些问题进行解释，也对 vgo  进行一些扩展。我写这篇文章不是为他说好话，我只是想试着解释一些大家对于 vgo 感觉到比较困惑的问题，我认为这些相对简单，而把更加困难的问题留给 Russ 集中精力去处理，这样当我有不懂的地方的时候，就知道去哪找答案了。

## 依赖包版本是如何确定的

虽然这一点在 [vgo 使用指南](https://research.swtch.com/vgo-tour) 中说的很清楚，但是这个问题还是成为了大家最困惑的地方。之所以这样，我觉得可能是大家看了 Russ 的文章以后，误解了他的意思，并且没有继续读 [vgo 使用指南](https://research.swtch.com/vgo-tour) 的原因，因为如果跟着指南亲自实践一下后，许多问题都会迎刃而解了。

其实我们只需要考虑三种情况，下面对这三种情况分别进行说明

### 添加新的依赖包

第一种情况非常简单，当在项目中引入一个新的包时，vgo 会默认使用这个包最新的版本，除非你手动指定了一个特定的版本。这么做合情合理，除了对于误认为 vgo 会默认使用包的最旧版本的人，这一点不会让任何人感到困惑。在 [vgo 使用指南](https://research.swtch.com/vgo-tour) 中对这一点有明确的说明。

### 依赖同一包的不同主版本

大部分包管理工具不允许这样。vgo 通过在包的引用路径上添加版本号的办法来实现引用一个包的不同版本，比如在一个新版本的引用路径上加上 /v2 （或者其它的版本号），这样一来，这个引用就变成了和其它版本完全无关的一个依赖包

现在我还不是太清楚，用户应该如何管理不同版本的路径问题。但是我想 Russ 会在接下来的一周对这一点进行一些说明。不管怎样，除非禁止这种行为，不然这就是处理引入一个包多个主版本的唯一的办法。所以我不认为这是一个特别需要我们关心的问题，对这一点的进一步讨论可以在等到有更多资料的时候再说

### 同一主版本下的多个次版本

假设主版本一样，最终使用的版本是最小版本集合中最高的版本。也就是说，如果项目中有 3 个模块依赖于 turtle 包，这样就构成了一个包含 3 个版本号的 一个最小版本集合，如下所示 

* v1.1.0
* v1.2.4
* v1.3.2
 
按照 [版本规范](https://semver.org/)，vgo 认为 v1.3.2 是这个版本集合中最高的版本，所以项目使用的就是这个版本

这个「最小版本」可能会让很多人误解，其实 Russ 想要表达的是：即使包 turtle 已经有 v1.3.3 版本， vgo 也不会使用，因为 v1.3.2 已经可以满足项目中三个模块的使用需求了。

Russ 不是在文章里说 vgo 总是倾向于使用「旧」版本吗？

在 Russ 的文章中，他多处用到了 「旧」 这个字，尤其是他的这句话：”因为不能发布旧版本“，让很多人误以为如果一个项目先发布了 v1.3.0， 之后又发布了 v1.2.2 ，vgo 就不能处理了。

这是不对的，不管是从我的亲自实践还是 Russ 在 [reddit](https://www.reddit.com/r/golang/comments/7yxlfz/researchrsc_go_package_versioning_go_versioning/dukkghy/) 上做的，都可以看到 vgo 可以应对这种情况

vgo 确定应该使用哪个版本是根据 [版本规范](https://semver.org/) 来的。也就是说，你可以在发布了 v1.3.0 之后再发布 v1.2.2，只是用户不能从 v1.3.0 「更新」到 v1.2.2，即使 v1.2.2 是后来发布的也不行， 因为根据 [版本规范](https://semver.org/) ，v1.2.2 版本比 v1.3.0 低。

## 更新依赖的方法

既然我们已经知道了 vgo 如何确定包版本，接下来让我们看看更新依赖包的一些方法

前面我们用到了 turtle 包的例子，所有的模块对 tuttle 的依赖构成下面这样一个最小版本集合

* v1.1.0
* v1.2.4
* v1.3.2

我们继续从这个例子出发，看看更新 turtle 包的一些方法

### 手动更新

如果想使用 turtle 包的 v1.4.4 版本，我们可以手动更新任何一个模块的 go.mod 文件，把依赖版本指定为 v1.4.4。vgo 提供了许多命令来实现诸如选择一个特定版本、更新到最新版本以及许多其它功能，用来辅助我们完成更新包版本的任务，可以查看 [vgo 使用指南](https://research.swtch.com/vgo-tour) 学习如何使用这些命令

### 添加或者更新依赖项

除了手动更改依赖项配置文件外，另外一个，也是唯一个可以让依赖项得到更新的办法就是添加一个新的包或者更新一个已经存在的包。这是因为一个包更新后，它所依赖的包也会更新。比如我们更新了包 foo （或者是新引入包 foo），从而改变了项目中 turtle 包的最小依赖集合， 而且从最小依赖集合中选出来的版本和之前的不一样，turtle 包就会被更新。

值得注意的是这种嵌套依赖可以有任意多层，比如包 foo 依赖包 bar，包 bar 又依赖包 spaz，直到有一个包最终依赖包 turtle@v1.3.6，这样就导致项目中 turtle 的版本从 v1.3.2 更新到了 v1.3.6，因为现在 v1.3.6 是满足所有模块构建需求的最小版本

## 潜在问题和相反的观点

既然我们已经知道了 vgo 如何选择包版本以及如何对包进行更新，接下来我们就看看大家对 vgo 最关心的一些问题

### 嵌套依赖包不会被更新

我在许多地方看到过这个说法，但这是不对的。 vgo 在确定依赖包版本时，会将项目中所有模块考虑在内。这也意味着，如果我们更新了包 foo，最终使用的包版本一定会等于或者高于包 foo 的 go.mod 文件中指定的版本

### 安全更新不是默认行为

对于安全更新，大家主要集中在这个基本问题上：如果我们使用的包版本是 v1.2.0，我们应该可以指定是要更新到兼容性比较好的 v1.2.x, 还是更新到任意的次版本 v1.X.Y

使用 vgo ，这种更新不会自动发生，除非使用了像 vgo get -u 这样的命令告诉 vgo 更新依赖包

就个建议确实有可取之外，但是我也不确定即可应该添加这样的功能，总之作为初学者，除了新建一个新的项目，基本不会更新依赖包。所以任何发布的新的依赖包版本是不会包含进已经部署的代码中去的，除非我们要发布项目的一个新版本。如果不是这样，在不同服务器上部署的代码使用的是不同的依赖包版本，那简直就是调试和管理的噩梦。

这样只有在我们开始构建的时候才需要去考虑这个问题，但其实这也不是一个特别大的问题。由于 vgo test all 的测试范围变小了，只测试单独的项目，我们就可以实现自动构建：先更新依赖包、然后测试，如果测试全部通过我们就部署。就像使用其它依赖管理工具所期望的那样。可以关注一下 vgo 之后是否会禁止这样做，即使不禁止，这也不应该是一个特别需要关注的主要问题。你可以自定义你的构建工具，但这并不意味着你的用户和你用的是同一套标准库工具

Russ Cox 在 [golang-nuts 邮件列表](https://groups.google.com/forum/#!topic/golang-nuts/jFPz5yZCPcQ) 的一个回复中也提到了这一点

> 在 [vgo 使用指南](https://research.swtch.com/vgo-tour) 中， 我提到了 「all」 被重新定义了，变得更加实用了。所以在尝试更新依赖包时，它是一个特别好用的工具，运行 go test all 后，如果通过了全部的测试而且你相信你写的测试用例，那么就保存对 go.mod 的修改。一些人甚至会构建一个 bot 来自动做这些事情。我不认为最小版本选择意味着你必须用旧版本，它只是不会自动这样做，除非你手动指定并且准备好去体验新版本的功能有多好。而不是运行了 ge get，然后所有版本一下就都更新了。

### 包会变得难以维护

问题是这样的：作为一个包管理工具，如果当包有新版本发布时，不能自动更新到这个版本，用户就不能使用新的功能

作为一种对比，即使包管理工具把依赖包升级到了最新版本，同样会导致一些问题。比如说我们的项目中引用了包 turtle, 而它依赖包 foo，当包 foo 发布了一个不兼容的新版本，这时包管理工具默认更新到了这个版本，这时项目构建一定会出现问题，当使用者遇到这个些问题时 ，肯定会向包 turtle 的作者提出修改意见，但是这只是因为它的依赖包 foo 被错误的更新导致的，包 turtlｅ不应该为此承担任何责任

这使我不禁要想，对于包的开发者来说哪种方式会带来更多的麻烦，是包管理工具默认把依赖包升级到最新版本，还是按 go.mod 中指定的版本来确定依赖包的版本

不管哪种方式，总会有一部分人使用的是旧版本，一部分人使的是最新的版本。在这一点上，我们还不能十分肯定的说哪种方式会导致更多的问题

我认为对于用户提出的这种问题 ，也不是特别难解决，我们可以让用户先更新包版本，这是很常见的做法，尤其是对于像 Atom 这样的桌面应用来说，这种做法是相当合理的

###  偶然的构建错误

假设我们的项目中用到了包 foo，包作者偶然发布了一个带有不兼容修改的版本 v1.3.2，但是很快又在之后的版本 v1.3.3 修复了这个 bug。如果 vgo 通过依赖关系确定 v1.3.2 是最小的依赖版本，但是这个版本确实会让我们的项目不能通过构建，这时候是不是自动使用最新的 v1.3.3 版本会更好呢？

这个情况比较特殊，而且很少发生，在开发过程中，我很少碰到这种问题。如果发生了这种情况，就是更新（或者添加）了一个依赖包，由于间接依赖的关系，这个更新导致对包 foo 的最小依赖版本变成 v1.3.2（那个包含了不兼容修改的版本）。大多数时候，包作者会很快发布一个新版本解决这个问题，所以依赖于那个短时间存在的不兼容的版本的几率很小。即使发生了，依赖错误版本的包也会进行适当的修改来以保证顺利构建

如果这个情况确实发生了，我们可以采取下面的办法来解决

1、告知那个包的作者，它的包依赖了一个不兼容的其它包，好让作者更新包的 go.mod 文件，改用正确的版本

2、更新你自己的 go.mod，把包 foo 的版本指定为 foo@v1.3.3

vgo 对这种情况的处理是非常自然的，只有当开发者手动指定时，才会发生。所以正常情况下不会遇到这种情况，只有进行更新时才会出现，这是发生这个问题的唯一逻辑时间点。在构建的时候，要么正好发生这个情况，要么这个问题已经被解决了。

### 离线构建

有些开发者关心离线构建的问题，我对此也有自己的疑惑， [ty64738 在 reddit 上提到了](https://www.reddit.com/r/golang/comments/7yxlfz/researchrsc_go_package_versioning_go_versioning/duknor4/)：现在 vgo 使用 $GOPATH/src/v 这个目录存放源码，一旦完成了对项目依赖包版本的解析，我们就可以用这个目录下的代码进行离线构建了。我不知道这是否是最终的方式，但是至少 Russ 认为这是一个合理的方式 

### 以 HTTP 来下载代码不安全

一些开发者认为 vgo 获取代码的方式不安全，因为它用的是 HTTP 而不是  HTTPS。不过据我所知，在 vgo 的源码实现里，应该采用的是 HTTPS 的方式， Russ 在文章中用 「HTTP」 可能只是用它来指代完成实际传输任务的 git 或者 bzr.

### API 的限制

一些开发者注意到 Github 提供的 API 有一些限制，导致必须做一些额外的工作才能让 vgo 正常工作， 我认为这种问题可以通过 Github 和 Google 之间的沟通得到解决

## 使用 vgo 的好处

写这篇文章主要是为了阐明和讨论大家关于 vgo 比较关心的一些问题，但是我觉得顺便提一下使用 vgo 的好处也是很有必要的。下面列出的肯定不是 vgo 全部的优点，只是我个人认为比较好的

### 大多数情况和之前一样正常

许多优秀的开发者聚在一起，开发了依赖包管理工具 dep，我非常喜欢这个工具，但是在熟练使用它之前需要一个学习的过程

相比来说，vgo 和现有的 go 命令行工具的使用方式几乎一样，可能唯一需要我们适应的就是 go.mod 这个文件了。但是如果这真的是从 go 过度到 vgo 最大的障碍的话，那么 vgo 就是非常成功的了


### go test all 变得更加实用了

vgo 也让 go test all 命令变得更加实用了，因为现在它只是测试当前模块和它的依赖项。虽然这不是开发 vgo 的主要目标，但这个变化绝对是深思熟虑的结果，我越来越喜欢这个命令了。

更进一步说，我希望这种不依赖 $GOPATH 的方式，可以让像 gorename 这样的程序 工作得更加一致，通过把作用范围限制在一个单独的模块，就不会像之前一样，只要 $GOPATH/src 目录下有一个不正常的模块就会导致 gorename 运行失败。此外由于单个模块构建时不依赖外部模块，构建失败也很少会发生。

### 构建应该更加一致

虽然我不能保证每欠构建都 100% 一致，但是通过使用 vgo， 我们在构建时，就可以避免上面我们提到的那种痛苦经历（当要构建时，依赖包 foo 刚刚发布一个新的版本），这并不意味着你不能使用最新版本的 foo，只是在你手动更新前不会自动更新到最新版本

正如 Russ 在文章里提到的

> 我不认为最小版本选择机制并不意味着你总是要使用旧的包版本，它只是不会自动这样做，除非你手动指定并且准备好去体验新版本的功能有多好。而不是运行了 ge get，然后所有版本一下就都更新了。

如果使用了一个可以自动更新依赖包版本的工具，任何开发者都会遇到由于版本更新带来的问题。有时候问题会变得更加严重，可能团队中的每个人都会被这样的问题困扰，当这种情况发生时，许多人会同时尝试解决同一个问题，这样是很浪费资源的。如果使用了 vgo，只有当某人更新包时才会出现，整个团队困扰于同一个更新，多个人同时解决同一个问题的事情就很少发生了


### 最大化的内部控制，最小化的外部控制

Russ 在 [golnag-nuts 邮件列表](https://groups.google.com/forum/#!topic/golang-nuts/jFPz5yZCPcQ) 里提到的另外一点是：通过使用 vgo，你可以完全控制自己的构建，减少对使用自己包的用户的的限制

Russ 是这样说的

> 是的，我会在明天的文章里详细阐述最小包版本选择的问题，我认为这是 vgo 里面最重要的算法策略细节。通过这个机制可以让我们完全控制自己的代码，对引用我们代码的用户只会有有限的限制

我想 Russ 会对这一点进行详细的说明，我对这一点的理解是：现在包的作者只能说 ‘我的代码只能在 foo@1.2.1 上正常运行’ 而不是说 ‘我的代码在 foo@v1.2.1 或者更高版本下都可以正常运行“， 后者通过其它的包管理工具是很容易做到的。

David Anderson 在邮件列表里对这样的经历有很好的描述

> 在我的 Go 项目中（尤其是那些使用了 Kubernetes 客户端库的项目，这个库对于 '作为库的作者，我可以对用户制定任何限制’ 这个原则应用得有点过了），对 glide 和 dep 的依赖给我带来了很多麻烦。换种说法就是：传统的 “任意版本选择” 算法对库的作者起到了错误的激励作用，他们可以随意发布版本，这样做不会给他们带来任何痛苦。但是对于引用他们代码的开发者来说，这种版本发布的随意性，可能会带来很大的痛苦。“不用负责的授权” 是许多激励机制失败的原因

----------------

via: https://www.calhoun.io/exploring-vgo/

作者：[Jon Calhoun](https://www.calhoun.io/about)
译者：[jettyhan](https://github.com/jettyhan)
校对：[polaris1119](https://github.com/polaris1119)

本文由 [GCTT](https://github.com/studygolang/GCTT) 原创编译，[Go 中文网](https://studygolang.com/) 荣誉推出