---
title: git submodule问题
url: 383.html
id: 383
categories:
  - Git
  - 小提示
date: 2017-07-30 12:34:31
tags:
---

有一个不在.gitmodule文件中的submodule在更新，解决办法就是

    git ls-files --stage | grep 160000
    

这可以看到所有的Submodule文件，然后

    git rm --cached PATH