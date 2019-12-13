---
title: bfg工具 清除git 历史记录
date: 2015/10/8
categories:
  - other
tags:
  - git
comments: true
abbrlink: 49197
img:
---


下载jar包 
[下载地址](https://search.maven.org/classic/remote_content?g=com.madgag&a=bfg&v=LATEST)


下载好后在你的根目录下执行
```
java -jar bfg.jar --strip-blobs-bigger-than 100M /你的项目地址
```

回到你的项目目录执行

```
git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

```
git push
```



