title: git-study
abbrlink: 2464661975
date: 2018-12-17 16:11:29
tags:
categories:
---
# 例子
```
victordeMacBook-Pro:demo victor$ find .git
.git
.git/ORIG_HEAD
.git/config 当前git仓库的配置文件。
.git/objects
.git/objects/02
.git/objects/02/91bf11241b54c5ca3941cd00674a619ada4ab7
.git/objects/ca
.git/objects/ca/953118bc85aaee49a38a3f2ed2b5e0f65065b6
.git/objects/pack
.git/objects/pack/pack-d0ce9798310cc6f86101f3a7d63e3f593094f954.idx
.git/objects/pack/pack-d0ce9798310cc6f86101f3a7d63e3f593094f954.pack
.git/objects/7c
.git/objects/7c/ece5fc917708caf1fd0c9bce6025af449b2102
.git/objects/info
.git/objects/info/packs
.git/objects/65
...


.git/HEAD
.git/info
.git/info/exclude
.git/info/refs
.git/logs
.git/logs/HEAD
.git/logs/refs
.git/logs/refs/heads
.git/logs/refs/heads/develop
.git/logs/refs/heads/open-source
.git/logs/refs/heads/master
.git/logs/refs/remotes
.git/logs/refs/remotes/origin
.git/logs/refs/remotes/origin/develop
.git/logs/refs/remotes/origin/open-source
.git/logs/refs/remotes/origin/master
.git/logs/refs/stash


.git/description   当前git仓库的描述。


.git/hooks
.git/hooks/commit-msg.sample
.git/hooks/pre-rebase.sample
.git/hooks/pre-commit.sample
.git/hooks/applypatch-msg.sample
.git/hooks/pre-receive.sample
.git/hooks/prepare-commit-msg.sample
.git/hooks/post-update.sample
.git/hooks/pre-applypatch.sample
.git/hooks/pre-push.sample
.git/hooks/update.sample


.git/refs
.git/refs/heads
.git/refs/tags
.git/refs/remotes
.git/refs/remotes/origin


.git/index
.git/packed-refs
.git/COMMIT_EDITMSG
.git/FETCH_HEAD
.git/sourcetreeconfig

```

## .git/config
```
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        ignorecase = true
        precomposeunicode = true
[branch "develop"]
[hubflow "branch"]
        master = master
        develop = develop
[hubflow "prefix"]
        feature = feature/
        release = release/
        hotfix = hotfix/
        support = support/
        versiontag = 
[remote "origin"]
        url = git@gitlab.demo.com:git/algorithm.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
[branch "open-source"]
        remote = origin
        merge = refs/heads/open-source

```

## git log
from-id to-id author <email> timestamp operation: comments



```
$ cat .git/logs//refs/heads/develop
255a1cf9335aca115baf3ad1edadb12b5be95fe0 cf4b1d712ff25a7f348b2c40238cbf074037cd23 victor <920443721@qq.com> 1540263345 +0800    pull git develop: Fast-forward
cf4b1d712ff25a7f348b2c40238cbf074037cd23 ee5021cb6c2133ca8a7a706b7042bbacef6cc957 victor <920443721@qq.com> 1540263844 +0800    merge origin/develop: Fast-forward
ee5021cb6c2133ca8a7a706b7042bbacef6cc957 52417cd30a21dc94d1b027b026c78af7a3822fa5 victor <920443721@qq.com> 1540352777 +0800    merge origin/develop: Fast-forward
52417cd30a21dc94d1b027b026c78af7a3822fa5 17f005f0a14d9229f9a5d5ac6074e62ccba70965 victor <920443721@qq.com> 1540376598 +0800    commit: when model submit ,return bdid and cid
17f005f0a14d9229f9a5d5ac6074e62ccba70965 3e1900ec1e90ed2da9e5958b25caecd0ff7556cf victor <920443721@qq.com> 1540894241 +0800    pull origin develop: Fast-forward
3e1900ec1e90ed2da9e5958b25caecd0ff7556cf 887616e4145dfc4cfda51f70e68f75426477d426 victor <920443721@qq.com> 1540894255 +0800    merge feature/bugfix_1024: Merge made by the 'recursive' strategy.
887616e4145dfc4cfda51f70e68f75426477d426 3f84beac47434a5cce2c74a67dcdff609cc556fa victor <920443721@qq.com> 1541647787 +0800    commit: move database to sdmk
3f84beac47434a5cce2c74a67dcdff609cc556fa d466b0bd4e5547763353b70e2b8d08439b2dec56 victor <920443721@qq.com> 1541989928 +0800    commit: revert to opal db
d466b0bd4e5547763353b70e2b8d08439b2dec56 c9fd7b00c102049f76ae79c6eac7c4cae3d28915 victor <920443721@qq.com> 1541989958 +0800    merge file-test: Merge made by the 'recursive' strategy.
c9fd7b00c102049f76ae79c6eac7c4cae3d28915 a6062bd7ff3f946f93eabf9780ed73c4130218b6 victor <920443721@qq.com> 1542003897 +0800    commit: config bugfix
a6062bd7ff3f946f93eabf9780ed73c4130218b6 9278cfa3e6475f780b66f9a9c84f516fb10bae42 victor <920443721@qq.com> 1542005375 +0800    commit: fix test
9278cfa3e6475f780b66f9a9c84f516fb10bae42 b1a09b2a675180fd7ee73a376638261c5da7500a victor <920443721@qq.com> 1542360229 +0800    merge feature/local-log: Fast-forward
b1a09b2a675180fd7ee73a376638261c5da7500a 3a4d38f926724afbf49b8a336150a5dc501aa849 victor <920443721@qq.com> 1542878535 +0800    merge origin/develop: Fast-forward

```


# git对象
## SHA
所有用来表示项目历史信息的文件,是通过一个40个字符的（40-digit）“对象名”来索引的，对象名看起来像这样:

6ff87c4664981e4397625791c8ea3bbb5f2279a3
你会在Git里到处看到这种“40个字符”字符串。每一个“对象名”都是对“对象”内容做SHA1哈希计算得来的，（SHA1是一种密码学的哈希算法）。这样就意味着两个不同内容的对象不可能有相同的“对象名”。

这样做会有几个好处：

- Git只要比较对象名，就可以很快的判断两个对象是否相同。
- 因为在每个仓库（repository）的“对象名”的计算方法都完全一样，如果同样的内容存在两个不同的仓库中，就会存在相同的“对象名”下。
- Git还可以通过检查对象内容的SHA1的哈希值和“对象名”是否相同，来判断对象内容是否正确。


## Blob 对象

新文件纳入到 Git 后会被五马分尸，它的内容被扔到在一个 blob 对象中，它的对象名是基于内容运算生成的一个 40个字符的 SHA1值。

blob 没有文件名，只有内容。

```
git cat-file blob HEAD:welcome.txt
Hello.
Nice to meet you.
```
在文件内容的前面加上blob 25<null>的内容，然后执行SHA1哈希算法。
```
$ ( printf "blob 25\000"; git cat-file blob HEAD:welcome.txt ) | sha1sum
fd3c069c1de4f4bc9b15940f490aeb48852f3c42  -
```

## Tree 对象

一个 tree 对象就是一大坨指针，指向：

- 其他 small tree（子级 tree）
- blob
可以把 Tree 对象想象为 Linux 文件系统中的目录，记录了子目录的信息、文件信息。

```
$ ( printf "tree 39\000"; git cat-file tree HEAD^{tree} ) | sha1sum
f58da9a820e3fd9d84ab2ca2f1b467ac265038f9  -
```

## Commit 对象

一个 commit 对象由以下几部分组成

- 作者
- 提交者
- 注释
- 指向一个 big tree 的指针

```
$ git cat-file commit HEAD
tree f58da9a820e3fd9d84ab2ca2f1b467ac265038f9
parent a0c641e92b10d8bcca1ed1bf84ca80340fdefee6
author Jiang Xin <jiangxin@ossxp.com> 1291022581 +0800
committer Jiang Xin <jiangxin@ossxp.com> 1291022581 +0800

which version checked in?
```

在提交信息的前面加上内容commit 234<null>（<null>为空字符），然后执行SHA1哈希算法。
```
$ ( printf "commit 234\000"; git cat-file commit HEAD ) | sha1sum
e695606fc5e31b2ff9038a48a3d363f4c21a3d86  -
```
## 对象之间的关系

![upload successful](/images/pasted-299.png)

## 分支
https://gotgit.readthedocs.io/en/latest/_images/git-repos-detail.png
![upload successful](/images/pasted-300.png)


## 对象存储




# git引用
你可以在 .git/refs 目录下找到这类含有 SHA-1 值的文件。 在目前的项目中，这个目录没有包含任何文件，但它包含了一个简单的目录结构：
```
$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/tags
```

## head引用
## tag引用
## remote引用
## stash引用

# 参考
http://gitbook.liuhui998.com/1_2.html
https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%AF%B9%E8%B1%A1
https://gotgit.readthedocs.io/en/latest/02-git-solo/030-head-master-commit-refs.html