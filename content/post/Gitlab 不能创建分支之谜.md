---
title: Gitlab不能创建分支之谜
date: 2021-07-08T04:58:51.213Z
authors:
  - lordran
tags:
  - Git
  - Gitlab
  - DevOps
categories:
  - 操作系统
---
- - -

今天遇到一件很奇怪的事情。

客户在使用我们的devops平台创建分支时一直提示： **Invalid reference name，**可是作为reference的master分支是存在的。我们代码管理使用的是CE版的Gitlab，所以我查询问题时想着是不是调用 Gitlab API时传入的参数不对，review代码几次没有发现可疑的地方，在举足无措时想到在Gitlab Web端创建分支。

{% asset_img 1.png %}

仍然提示"**Invalid reference name"，**不过比API好的是显示了分支名：**bug/v1.0.1.1-hotfix，**不过这跟平台所给的或者Git中的概念不一致**，\*\***bug/v1.0.1.1-hotfix**应该算是要创建的分支名**，master**才算是**reference。\*\*这里的提示很容易误导。暂时鄙视一下Gitlab，API确实不怎么好用 。

虽然知道了问题出在Gitlab而非devops平台代码，但是也不知道具体原因。
之后再次选择在本地clone 仓库，然后创建分支：

{% asset_img 2.png %}

这时突然想起.git文件夹中，文件的存放结构：

{% asset_img 3.png %}

而对于分支名**bug/v1.0.1.1-hotfix**在上图中的**bug**所在目录创建同名的**bug**目录**，**这受到了操作系统的限制，导致出现上诉的问题。

最后再吐槽下gitlab的API。