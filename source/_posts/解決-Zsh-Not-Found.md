---
title: '解決 Zsh Not Found '
date: 2019-05-26 02:30:49
tags:
---

在 Terminal 打 
`printenv`
Shell 屬性為
`SHELL=/bin/zsh`

查看 目前Env 的變數  `echo $PATH`
編輯 .zshrc  加入路徑
`export PATH=$PATH:路徑名稱`
最後在 `source .zshrc` 檔，或是重新啟用 終端機就可以了

