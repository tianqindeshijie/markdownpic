## Git Flow的使用实践
我们当前的源代码管理和版本管理存在很大的问题，新功能开发版本代码，测试版本代码，正式版本代码，bug修复版本代码都没有一个明确的分支体系和正规的操作流程。所以造成了现在混乱的代码管理情况，所有代码（不管是开发的，测试的，以及bug修复等）都在一个Mater分支进行，有时候正式版本需要进行bug修复的时候完全理不清哪些是需要进行升级的，哪些不需要。针对这种情况我觉得团队里应该制定一套完整的源代码版本管理规范。

### 版本管理的挑战
我们当前的源代码都是基于Git来进行管理的，Git本身有很多优点，Git的优点这里就不做过多的介绍了，下面说说我们基于Git做版本管理容易出现的问题：

1. 自己开始一个功能的开发，如何不影响其他人的功能开发？
2. Git由于很容易创建新分支，分支多了如何管理，时间久了，如何知道每个分支是干什么的？
3. 哪些分支已经合并回了主干？
4. 如何进行Release的管理？开始一个Release的时候如何冻结Feature, 如何在Prepare Release的时候，开发人员可以继续开发新的功能？
5. 线上代码出Bug了，如何快速修复？而且修复的代码要包含到开发人员的分支以及下一个Release?

大部分开发人员现在使用Git就只是用三个甚至两个分支，一个是Master, 一个是Develop, 还有一个是基于Develop打得各种分支。这个在小项目规模的时候还勉强可以支撑，因为很多人做项目就只有一个Release, 但是人员一多，而且项目周期一长就会出现各种问题。

### Git Flow
针对前面的问题，我们引入一个基于Git的工作模式GitFlow。通过这个模式可以很好的解决前面所说的问题。下面我们将具体的介绍一下Git Flow。

Gitflow（git工作流）定义了一个围绕项目发布的严格分支模型。它提供了用于一个健壮的用于管理大型项目的框架。

Gitflow（git工作流）没有用超出功能分支工作流的概念和命令，而是为不同的分支分配一个很明确的角色，并定义分支之间如何和什么时候进行交互。除了使用功能分支，在做准备、维护和记录发布也使用各自的分支。当然你可以用上功能分支工作流所有的好处：Pull Requests、隔离实验性开发和更高效的协作。

可能通过上面的解释大家还不是很清楚。下面我们根据图给大家进一步的解释。
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/1.jpg)
Git Flow常用的分支

**Master分支**
这个分支始终保持可以随时发布的状态，在Master分支各个版本通过tag来标记。上图里的v0.1和v0.2和v1.0就是tag。 这个分支只能从其他分支合并，任何开发人员不能直接修改Master分支。

**Develop 分支**
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/2.png)
develop分支是开发过程中代码中心分支。与master 分支一样，这个分支也不允许开发者直接进行修改和提交。
程序员要以develop分支为起点新建feature 分支，在feature 分支中进行新功能的开发或者代码的修正。也就是说develop分支维系着开发过程中的最新代码，以便程序员创建feature分支进行自己的工作。
**Feature 分支**
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/3.png)
这个分支主要是用来开发一个新的功能，该分支以develop分支为起点，是开发者直接更改代码发送提交的分支。开发流程：
1.  从develop分支创建feature分支
2. 从feature分支中实现目标功能
3. 通过Github 向develop发送pull request
4. 接受其他开发者审核后，将Pull Request合并至develop分支
5. 与develop分支合并后，已经完成工作的feature分支可以在适当的时机删除

> **注意:**当feature开发完毕后，要合并回develop分支。feature分支永远不会和master分支打交道。

**Release分支**
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/4.png)
当你需要一个发布一个新Release的时候，我们基于Develop分支创建一个Release分支，完成Release后，我们合并到Master和Develop分支。在这个分支，我们只处理与发布前准备相关的提交，比如版本编号变更的元数据的添加工作。

> **注意：**我们可以在这个Release分支上测试，修改Bug等。一旦打了Release分支之后不要从Develop分支上合并新的改动到Release分支，该分支绝对不能包含需求变更或者功能变更等重大修正。这一阶段的提交数应该限制到最低。

**Hotfix分支**
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/5.png)
当我们在Master发现新的Bug时候，我们需要创建一个Hotfix, 完成Hotfix后，我们合并回Master和Develop分支，所以Hotfix的改动会进入下一个Release。Hotfix开发完后需要合并回Master和Develop分支，同时在Master上打一个tag。下述情况需要创建 Hotfix分支：
- Master版本中发现了bug 或者漏洞
- develop 分支正在开发新功能，无法面向用户进行发布
- 漏洞需要及早处理，无法等到下一次版本发布

### 如何使用Git Flow
上面说了很多都是介绍git flow的但是对于git Flow的使用可能还是一头雾水。的确前面的介绍我git Flow如果让大家直接用命令行操作的话，无疑是非常复杂的。除非是非常熟悉git和git flow的人员，否则全凭git命令行想要完成整个git flow的流程是非常麻烦的。当然既然我们决定要使用git flow来规范我们的源代码管理，那就必然有适合大家的简单实现方式，下面就介绍几种：
####  SourceTree
#####  sourceTree介绍
SourceTree是一款优秀的git 客户端工具软件，有着优秀的可视化界面，容易操作和上手。当然我们使用SourceTree是因为它集成了Git Flow功能，下面将具体的介绍如何使用它的Git Flow功能：
1. git flow 初始化

先点击下Git工作流按钮(first blood), 初始化下环境. 这样就有了master 和 develop 两个分支. 还为功能分支, 发布分支, 补丁分支分别制定了前缀.初始化直接使用默认就可以了。
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/6.png)

2. 开发进行功能
当需要开发一个新功能的时候. 切换到 develop 分支上, 接着还是点击 Git工作流按钮(double kill), 选择建立新的功能, 然后为功能起一个名称b. 这样本地分支目录下就会多了一个feature文件夹, 里面有一个b分支.
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/7.png)
当这个功能b开发完成后, 还是点击Git工作流按钮(trible kill), 点完成当前版本.代码会自动合并到developer分支上, 并且可以选择删除当前的b分支.

![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/8.png)

3. 发布新版本
当需要发布的时候, 切换到developer分支上, 点击Git工作流按钮(urltra kill), 选择建立新的发布版本1.2. 这样本地分支目录下就会多了一个release文件夹, 里面有一个1.2分支. 
当1.2没问题后, 还是点击Git工作流按钮(rammpage), 点完成当前版本, 给当前版本打个tag. 代码会自动合并到master分时和developer分支上.	
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/9.png)

4.Bug修复

如果线上版本一步小心有了一个bug, 切换到master分支上, 还是点击Git工作流按钮, 点建立新的修复版本1.1.1. 完成后还是点击Git工作流按钮, 点完成当前版本, 给当前版本打个tag. 代码会自动合并到master分时和developer分支上.

![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/10.png)

#### Git Flow命令行方式
##### git flow安装
如果想要使用命令行方式进行git flow操作的话，需要先安装git flow工具，安装的过程如下：
①首先要先安装Git。Git的安装前面已经有过介绍，这里就再多说了。
②https://github.com/nvie/gitflow/wiki/Windows

![111](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/11.png)
点击上图中的超链接，下载安装Git Flow需要三个文件(getopt.ext , libintl3.dll , libiconv2.dll)。下载完成之后将
这三个文件拷贝到git安装目录的bin文件夹下。
③打开Git Bash。然后在命令窗口输入：
```
git clone --recursive git://github.com/nvie/gitflow.git
```
此时在当前路径下就会生成一个gitflow文件夹。如果想执行安装路径，可以如下：
```
 git clone --recursive git://github.com/nvie/gitflow.git d:\gitflow
```
④文件复制完成之后，打开dos命令窗口，进入gitflow\contrib路径下，
执行：msysgit-install.cmd，如果git不是安装
在当前目录下，在安装gitflow的时候，后面需要加上git的安装路径。
msysgit-install.cmd "d:\git"
⑤返回Git Bash命令窗口，执行git flow,输出如下：
  ![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/12.png)
    
说明Git Flow已经安装成功。前面生成的gitflow文件已经完成了它的使命，可以删除了。
⑥执行git flow初始化，执行git flow int

##### 使用

- 初始化: git flow init
- 开始新Feature: git flow feature start MYFEATURE
- Publish一个Feature(也就是push到远程): git flow feature publish MYFEATURE
- 获取Publish的Feature: git flow feature pull origin MYFEATURE
- 完成一个Feature: git flow feature finish MYFEATURE
- 开始一个Release: git flow release start RELEASE [BASE]
- Publish一个Release: git flow release publish RELEASE
- 发布Release: git flow release finish RELEASE
- 开始一个Hotfix: git flow hotfix start VERSION [BASENAME]
- 发布一个Hotfix: git flow hotfix finish VERSION
####IDEA插件方式
##### 安装
idea的git flow插件安装非常简单，可以直接在idea的插件库里面去找
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/13.png)

不过这里说明一下，如果IDEA是2017版本的可以使用最新的插件，如果是2016版本的可以选用v0.4版本的，否则会出错。
安装后又下角会出现`Git Flow`的字符
##### 使用

①初始化
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/14.png)
这个里面的流程和sourceTree里面的有些区别，多了几个分支功能，不过我们暂时用不到这么多，后面的应该以sourceTree为准，不过多几个并不影响任何开发。

② 新增功能分支
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/15.png)

![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/16.png)

![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/17.png)

③新增Release
![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/18.png)

![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/19.png)


④ 新增hotfix

![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/20.png)

![enter image description here](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/git_flow_introduction/21.png)

#### Git Flow常见的使用错误
1. 出现：`fatal: Working tree contains unstaged changes. Aborting.`,这个是因为本地还有没有提交的代码，提交一下这个错误就不会出现了。
2. 出现：`Production and integration branches should differ.`这是因为我在初始化gitflow的时候branch name for production releases和branch name for “next release” development选的是同一个分支，将next release分支改为develop分支就好了 


####Git Flow规范
git flow指定了一个使用Git的工作模式，然后每个团队的情况都不尽相同，所以根据根据Git Flow在制定一套适合我们团队使用的规范才是最终的目标。下面就是指定团队内部一些约定规范（规范会根据大家的使用情况持续更新，大家可以不断的提出意见，直到非常稳定为止）：

1. master分支是主分支一直保持稳定，任何开发人员不允许直接往这个分支提交代码，只允许往这个分支发起merge request，只允许release分支和hotfix分支进行合流
2. develop下也不能直接提交开发代码，只允许feature分支和hotfix分支通过merge的方式合并代码。
3. 新功能的开发可以几个协同开发的人员公用一个feature分支，这样方便统一调试，避免了一直合并的问题。
4. 所有最后需要合并到master的分支必须做交叉的代码走查（code review），提高协同开发人员的代码交流，及时发现不合理的代码和问题。
5.