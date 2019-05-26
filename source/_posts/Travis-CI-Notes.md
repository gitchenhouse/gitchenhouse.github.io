---
title: 'Travis CI Notes '
date: 2019-05-26 02:28:57
tags:
---
### 加密私鑰
公鑰文件配置好了，然後就要配置私鑰文件，在hexo項目下面建立一個.travis的文件夾來放置需要配置的文件。

首先要安裝travis命令行工具 。
`$ gem install travis`
用命令行工具登錄：
`$ travis login --auto`
然後將剛剛生成的id_rsa複製到.travis文件夾，用命令行工具進行加密：
` travis encrypt-file id_rsa --add`
這個時候會生成加密之後的秘鑰文件id_rsa.enc，原來的文件id_rsa就可以刪掉了。

這時可以看到終端輸出了一段
`openssl aes-256-cbc -K $encrypted_xxxxxxxxxxx_key -iv $encrypted_xxxxxxxxxxx_iv`

這樣格式的信息，這是travis用來解密id_rsa.enc的key，先保存起來，後面配置.travis.yml會用到它。

為了讓git默認連接SSH還要創建一個ssh_config文件。在.travis文件夾下創建一個ssh_config文件，輸入以下內容：

```
Host github.com
    User git
    StrictHostKeyChecking no
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
```