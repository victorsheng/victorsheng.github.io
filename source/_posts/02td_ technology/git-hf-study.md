---
title: git-hf-study
abbrlink: 4036828974
date: 2018-11-21 16:49:23
tags:
categories:
---
# gitflow
http://datasift.github.io/gitflow/IntroducingGitFlow.html

# 工具下载
https://github.com/datasift/gitflow


# hf简介
Available subcommands are:
   init      Initialize a new git repo with support for the branching model.
   feature   Manage your feature branches.
   release   Manage your release branches.
   hotfix    Manage your hotfix branches.
   push      Push the changes from your current branch (plus any new tags) back upstream.
   pull      Pull upstream changes down into your master, develop, and current branches.
   update    Pull upstream changes down into your master and develop branches.
   version   Shows version information.

# git hf feature start feature1

Summary of actions:
- A new branch 'feature/feature1' was created, based on 'develop'
- The branch 'feature/feature1' has been pushed up to 'origin/feature/feature1'
- You are now on branch 'feature/feature1'



# git hf feature finish feature1

Summary of actions:
- The latest changes from 'origin' were merged into 'master' and 'develop'
- The feature branch 'feature/feature1' was merged into 'develop'
- Feature branch 'feature/feature1' has been removed
- Feature branch 'origin/feature/feature1' has been removed
- You are now on branch 'develop'

# git hf release start release1

Summary of actions:
- A new branch 'release/release1' was created, based on 'develop'
- The branch 'release/release1' has been pushed up to 'origin/release/release1'
- You are now on branch 'release/release1'

# git hf release finish release1

Summary of actions:
- Latest objects have been fetched from 'origin'
- Release branch has been merged into 'master'
- The release was tagged 'release1'
- Tag 'release1' has been back-merged into 'develop'
- Branch 'master' has been back-merged into 'develop'
- Release branch 'release/release1' has been deleted
- 'develop', 'master' and tags have been pushed to 'origin'
- Release branch 'release/release1' in 'origin' has been deleted


# git hf hotfix start fix1
Summary of actions:
- A new branch 'hotfix/fix1' was created, based on 'master'
- The branch 'hotfix/fix1' has been pushed up to 'origin/hotfix/fix1'
- You are now on branch 'hotfix/fix1'


# git hf hotfix finish fix1
Summary of actions:
- Latest objects have been fetched from 'origin'
- Hotfix branch has been merged into 'master'
- The hotfix was tagged 'fix1'
- Hotfix branch has been back-merged into 'develop'
- Hotfix branch 'hotfix/fix1' has been deleted
- 'develop', 'master' and tags have been pushed to 'origin'


# git hf update
Summary of actions:
- Any changes to branches at origin have been downloaded to your local repository
- Any branches that have been deleted at origin have also been deleted from your local repository
- Any changes from origin/master have been merged into branch 'master'
- Any changes from origin/develop have been merged into branch 'develop'
- Any resolved merge conflicts have been pushed back to origin
- You are now on branch 'develop'


# 比较
## git flow
Git flow的优点是清晰可控，缺点是相对复杂，需要同时维护两个长期分支。大多数工具都将master当作默认分支，可是开发是在develop分支进行的，这导致经常要切换分支，非常烦人。

更大问题在于，这个模式是基于"版本发布"的，目标是一段时间以后产出一个新版本。但是，很多网站项目是"持续发布"，代码一有变动，就部署一次。这时，master分支和develop分支的差别不大，没必要维护两个长期分支。
http://www.ruanyifeng.com/blog/2015/12/git-workflow.html


## Github flow
Github flow 是Git flow的简化版，专门配合"持续发布"。它是 Github.com 使用的工作流程。

Github flow 的最大优点就是简单，对于"持续发布"的产品，可以说是最合适的流程。

问题在于它的假设：master分支的更新与产品的发布是一致的。也就是说，master分支的最新代码，默认就是当前的线上代码。

可是，有些时候并非如此，代码合并进入master分支，并不代表它就能立刻发布。比如，苹果商店的APP提交审核以后，等一段时间才能上架。这时，如果还有新的代码提交，master分支就会与刚发布的版本不一致。另一个例子是，有些公司有发布窗口，只有指定时间才能发布，这也会导致线上版本落后于master分支。

上面这种情况，只有master一个主分支就不够用了。通常，你不得不在master分支以外，另外新建一个production分支跟踪线上版本。


## Gitlab flow
### 上游优先
Gitlab flow 的最大原则叫做"上游优先"（upsteam first），即只存在一个主分支master，它是所有其他分支的"上游"。只有上游分支采纳的代码变化，才能应用到其他分支。
### 持续发布
于"持续发布"的项目，它建议在master分支以外，再建立不同的环境分支。比如，"开发环境"的分支是master，"预发环境"的分支是pre-production，"生产环境"的分支是production。

开发分支是预发分支的"上游"，预发分支又是生产分支的"上游"。代码的变化，必须由"上游"向"下游"发展。比如，生产环境出现了bug，这时就要新建一个功能分支，先把它合并到master，确认没有问题，再cherry-pick到pre-production，这一步也没有问题，才进入production。

只有紧急情况，才允许跳过上游，直接合并到下游分支。

### 版本发布
对于"版本发布"的项目，建议的做法是每一个稳定版本，都要从master分支拉出一个分支，比如2-3-stable、2-4-stable等等。

以后，只有修补bug，才允许将代码合并到这些分支，并且此时要更新小版本号。
