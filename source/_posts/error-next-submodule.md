---
title: hexo next主题保存
date: 2017-11-30 22:41:04
tags:
categories: 异常
---
# hexo next主题保存，通过git的submodule功能对博客内容和主题分别进行版本控制

# 方案
https://stackoverflow.com/questions/12898278/issue-with-adding-common-code-as-git-submodule-already-exists-in-the-index

```
git ls-files --stage


git rm --cached themes/next

git submodule add git@github.com:victorsheng/hexo-theme-next.git themes/next
```