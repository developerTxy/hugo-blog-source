---
title: "git撤销命令"
date: 2023-12-24T15:32:46+08:00
lastmod: 2023-12-24T15:32:46+08:00
author: ["无涯·学者"]

categories:
- tech
- git

tags:
- git
- 撤销代码提交

keywords:
- git
- reset
- 撤销

description: "git 各种情况下的撤销操作" # 文章描述，与搜索优化相关
summary: "git 撤销" # 文章简单描述，会展示在主页
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
mermaid: true
cover:
    image: ""
    caption: ""
    alt: ""
    relative: false
---


 修改了本地的代码，然后使用：
git add file git commit -m '修改原因'
执行commit后，还没执行push时，想要撤销这次的commit，该怎么办？
解决方案： 使用命令：
git reset --soft HEAD^
这样就成功撤销了commit，如果想要连着add也撤销的话，--soft改为--hard（删除工作空间的改动代码）。
命令详解：
HEAD^ 表示上一个版本，即上一次的commit，也可以写成HEAD~1 如果进行两次的commit，想要都撤回，可以使用HEAD~2
--soft 不删除工作空间的改动代码 ，撤销commit，不撤销git add file
--hard 删除工作空间的改动代码，撤销commit且撤销add
另外一点，如果commit注释写错了，先要改一下注释，有其他方法也能实现，如：
git commit --amend 这时候会进入vim编辑器，修改完成你要的注释后保存即可。
为了搞清楚原理，我画流程图，找到了一个好的工具：https://www.draw.io/
#### git的工作流
工作区：即自己当前分支所修改的代码，git add xx 之前的！不包括 git add xx 和 git commit xxx 之后的。
暂存区：已经 git add xxx 进去，且未 git commit xxx 的。
本地分支：已经git commit -m xxx 提交到本地分支的。
![](https://cdn.nlark.com/yuque/0/2023/jpeg/12487912/1703246237651-1126389f-31ad-42b9-943d-a0242a3de06c.jpeg#averageHue=%23f6f6f6&clientId=u432bbeaf-14f6-4&from=paste&id=u81f5fa24&originHeight=1166&originWidth=802&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ud2019a73-8f72-472d-87cc-c03d67f1f31&title=)
基本原理如下：
![](https://cdn.nlark.com/yuque/0/2023/jpeg/12487912/1703246237658-a60219d4-46f9-402a-9e66-e7419f724b0e.jpeg#averageHue=%23f6f6f6&clientId=u432bbeaf-14f6-4&from=paste&id=u760098fc&originHeight=829&originWidth=1080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ue2a2a86b-6254-4bc3-8cde-c660917ed88&title=)
#### 代码回滚
在上传代码到远程仓库的时候，不免会出现问题，任何过程都有可能要回滚代码：
1、在工作区的代码

```javascript
git checkout -- a.txt   # 丢弃某个文件，或者
git checkout -- .       # 丢弃全部
```
注意：git checkout – . 丢弃全部，也包括：新增的文件会被删除、删除的文件会恢复回来、修改的文件会回去。这几个前提都说的是，回到暂存区之前的样子。对之前保存在暂存区里的代码不会有任何影响。对commit提交到本地分支的代码就更没影响了。当然，如果你之前压根都没有暂存或commit，那就是回到你上次pull下来的样子了。
2、代码git add到缓存区，并未commit提交

```javascript
git reset HEAD .  或者
git reset HEAD a.txt
```
这个命令仅改变暂存区，并不改变工作区，这意味着在无任何其他操作的情况下，工作区中的实际文件同该命令运行之前无任何变化
3、git commit到本地分支、但没有git push到远程

```javascript
git log # 得到你需要回退一次提交的commit id
git reset --hard <commit_id>  # 回到其中你想要的某个版
或者
git reset --hard HEAD^  # 回到最新的一次提交
或者
git reset HEAD^  # 此时代码保留，回到 git add 之前
```


4、git push把修改提交到远程仓库 1）通过git reset是直接删除指定的commit

```javascript
git log # 得到你需要回退一次提交的commit id
git reset --hard <commit_id>
git push origin HEAD --force # 强制提交一次，之前错误的提交就从远程仓库删除
```


2）通过git revert 用一次新的commit来回滚之前的commit

```javascript
git log # 得到你需要回退一次提交的commit id
git revert <commit_id>  # 撤销指定的版本，撤销也会作为一次提交进行保存
```


3） git revert 和 git reset的区别

- git revert 用一次新的commit来回滚之前的commit，此次提交之前的commit都会被保留；
- git reset 回到某次提交，提交及之前的commit都会被保留，但是此commit id之后的修改都会被删除

开发过程中，你肯定会遇到这样的场景：
场景一：
糟了，我刚把不想要的代码，commit到本地仓库中了，但是还没有做push操作！
场景二：
彻底完了，刚线上更新的代码出现问题了，需要还原这次提交的代码！
场景三：
刚才我发现之前的某次提交太愚蠢了，现在想要干掉它！
撤销 上述场景一，在未进行git push前的所有操作，都是在“本地仓库”中执行的。我们暂且将“本地仓库”的代码还原操作叫做“撤销”！
情况一：文件被修改了，但未执行git add操作(working tree内撤销)

```javascript
git checkout fileName
git checkout .
```


情况二：同时对多个文件执行了git add操作，但本次只想提交其中一部分文件

```javascript
$ git add *
$ git status
```


## 取消暂存

```javascript
$ git reset HEAD <filename>
```


情况三：文件执行了git add操作，但想撤销对其的修改（index内回滚）

```javascript
# 取消暂存
git reset HEAD fileName
# 撤销修改
git checkout fileName
```


情况四：修改的文件已被git commit，但想再次修改不再产生新的Commit

```javascript
# 修改最后一次提交
$ git add sample.txt
$ git commit --amend -m"说明"
```


情况五：已在本地进行了多次git commit操作，现在想撤销到其中某次Commit

```javascript
git reset [--hard|soft|mixed|merge|keep] [commit|HEAD]
```


#### 回滚
上述场景二，已进行git push，即已推送到“远程仓库”中。我们将已被提交到“远程仓库”的代码还原操作叫做“回滚”！注意：对远程仓库做回滚操作是有风险的，需提前做好备份和通知其他团队成员！
如果你每次更新线上，都会打tag，那恭喜你，你可以很快的处理上述场景二的情况

```javascript
git checkout <tag>
如果你回到当前HEAD指向

git checkout <branch_name>
```


情况一：撤销指定文件到指定版本

```javascript
# 查看指定文件的历史版本
git log <filename>
# 回滚到指定commitID
git checkout <commitID> <filename>
```


情况二：删除最后一次远程提交
方式一：使用revert

```javascript
git revert HEAD
git push origin master
```


方式二：使用reset

```javascript
git reset --hard HEAD^
git push origin master -f
```


二者区别：
revert是放弃指定提交的修改，但是会生成一次新的提交，需要填写提交注释，以前的历史记录都在； reset是指将HEAD指针指到指定提交，历史记录中不会出现放弃的提交记录。 情况三：回滚某次提交

```javascript
# 找到要回滚的commitID
git log
git revert commitID
删除某次提交
git log --oneline -n5


git rebase -i "commit id"^
注意：需要注意最后的^号，意思是commit id的前一次提交

git rebase -i "5b3ba7a"^
```


代码提交，谁都不希望撤销或者回滚，有时候又迫不得已。掌握这些命令，可以提交代码无忧！
