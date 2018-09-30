---
title: hexo next主题保存
tags:
  - hexo
  - git
  - submodule
categories:
  - 异常
abbrlink: 3858977206
date: 2017-11-30 22:41:00
---
# 通过git的submodule功能对博客内容和主题分别进行版本控制

# 方案
https://stackoverflow.com/questions/12898278/issue-with-adding-common-code-as-git-submodule-already-exists-in-the-index

```
git ls-files --stage


git rm --cached themes/next

git submodule add git@github.com:victorsheng/hexo-theme-next.git themes/next

git add .gitmodules
```

<img src="http://pic.victor123.cn/17-11-30/78270359.jpg">

## .gitmodulesn内容如下：
[submodule "themes/next"]
	path = themes/next
	url = git@github.com:victorsheng/hexo-theme-next.git
