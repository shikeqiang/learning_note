# 一、基础

## 1.常用命令

![Git å¸¸ç¨å½ä"¤](/Users/jack/Desktop/md/images/3054153c1904dfd5f3b9d3fbc6bf2375.jpeg)

- `git init`：创建 Git 库。

  - `git status` ：查看当前仓库的状态。

- `git show` ：# 显示某次提交的内容 git show $id

- `git diff` ：查看本次修改与上次修改的内容的区别。

- `git add <file>` ：把现在所要添加的文件放到暂存区中。

  - `git log -p <file>` ：查看每次详细修改内容的 diff 。

  - `git rm <file>` ：从版本库中删除文件。

  - `git reset <file>` ：从暂存区恢复到工作文件。

  - `git reset HEAD^` ：恢复最近一次提交过的状态，即放弃上次提交后的所有本次修改` 。

    > HEAD 本身是一個游标，它通常會指向某一个本地端分支或是其它 commit，所以你也可以把 HEAD 当做是目前所在的分支（current branch）。 可参见 [《Git 中 HEAD 是什么东西》](https://juejin.im/entry/59a38c5d6fb9a0248e5cc884) 。

- `git commit` ：把 Git add 到暂存区的内容提交到代码区中。

- `git clone` ：从远程仓库拷贝代码到本地。

- git branch：查看当前的分支名称。

  - `git branch -r` ：查看远程分支。

- `git checkout` ：切换分支。

- `git merge <branch>` ：将 branch 分支合并到当前分支。

- git stash：暂存。

  - `git stash pop` ：恢复最近一次的暂存。

- git pull：抓取远程仓库所有分支更新并合并到本地。

  - `git push origin master` ：将本地主分支推到远程主分支。

![整体流程](/Users/jack/Desktop/md/images/033f8d41d6f67a01a7cfefa6b9aa4cf4.png)

​	IDEA Git 插件越来越强大，很多时候，我们日常使用 Git ，更多使用它。具体的教程，可以看看 [《IntelliJ IDEA 下的使用 git》](https://blog.csdn.net/huangfan322/article/details/53220060)

## 2.Git和Svn的比较

Git 是分布式版本控制系统，SVN 是集中式版本控制系统。

1）SVN 的优缺点

- 优点
  - 1、管理方便，逻辑明确，符合一般人思维习惯。
  - 2、易于管理，集中式服务器更能保证安全性。
  - 3、代码一致性非常高。
  - 4、适合开发人数不多的项目开发。
- 缺点
  - 1、服务器压力太大，数据库容量暴增。
  - 2、如果不能连接到服务器上，基本上不可以工作，因为 SVN 是集中式服务器，如果服务器不能连接上，就不能提交，还原，对比等等。
  - 3、不适合开源开发（开发人数非常非常多，但是 Google App Engine 就是用 SVN 的）。但是一般集中式管理的有非常明确的权限管理机制（例如分支访问限制），可以实现分层管理，从而很好的解决开发人数众多的问题。

2）Git 优缺点

- 优点
  - 1、适合分布式开发，强调个体。
  - 2、公共服务器压力和数据量都不会太大。
  - 3、速度快、灵活。
  - 4、任意两个开发者之间可以很容易的解决冲突。
  - 5、离线工作。
- 缺点
  - 1、学习周期相对而言比较长。
  - 2、不符合常规思维。
  - 3、代码保密性差，一旦开发者把整个库克隆下来就可以完全公开所有代码和版本信息。

所以，很多公司的开发团队使用 Git 作为版本管理，而产品团队使用 SVN 。

## 3.常用操作

### 创建分支步骤

- 1、`git branch xxx_dev` ：创建名字为 `xxx_dev` 的分支。
- 2、`git checkout xxx_dev` ：切换到名字为 `xxx_dev` 的分支。
- 3、`git push origin xxx_dev` ：执行推送的操作，完成本地分支向远程分支的同步。

更详细的，可以看看 [《Github 创建新分支》](https://blog.csdn.net/top_code/article/details/51931916) 文章。

### tag

tag ，指向一次 commit 的 id ，通常用来给分支做一个标记。

- 打标签 ：`git tag -a v1.01 -m "Release version 1.01"` 。
- 提交标签到远程仓库 ：`git push origin --tags` 。
- 查看标签 ：`git tag` 。
- 查看某两次 tag 之间的 commit ：`git log --pretty=oneline tagA..tagB` 。
- 查看某次 tag 之后的 commit ：`git log --pretty=oneline tagA..` 。

修改**提交时写错的 commit 信息**，可以通过 `git commit --amend` 来对本次 commit 进行修改。

















































参照：芋道源码