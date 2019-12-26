---
title: git 忽略已经提交和没提交的文件
date: 2019/10/8
categories:
  - git
abbrlink: 29383
tags:
---

# 没有提交的

其实一直都知道有这么个功能 但是一直都是手写没有记录 

可以先去看下github官方提供的gitignore  里面有各种语言的 gitignore

https://github.com/github/gitignore



自己个性化的话 可以在上面的文件基础上修改

在Git工作区的根目录下创建一个特殊的 .gitignore

下面是我自己使用的
```

/target/**
/target
target
!.mvn/wrapper/maven-wrapper.jar

###STS###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

###IntelliJIDEA###
.idea
*.iws
*.iml
*.ipr

###NetBeans###
/nbproject/private/
/build/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/

#Compiledclassfile
*.class

#Logfile
*.log

#BlueJfiles
*.ctxt

#MobileToolsforJava(J2ME)
.mtj.tmp/

#PackageFiles#
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
.pdf
.zip
.doc

```


下面是一些语法使用 
```
• bin/: 忽略当前路径下的bin文件夹，该文件夹下的所有内容都会被忽略，不忽略 bin 文件

• /bin: 忽略根目录下的bin文件

• /*.c: 忽略 cat.c，不忽略 build/cat.c

• debug/*.obj: 忽略 debug/io.obj，不忽略 debug/common/io.obj 和 tools/debug/io.obj

• **/foo: 忽略/foo, a/foo, a/b/foo等

• a/**/b: 忽略a/b, a/x/b, a/x/y/b等

• !/bin/run.sh: 不忽略 bin 目录下的 run.sh 文件

• *.log: 忽略所有 .log 文件

• config.php: 忽略当前路径的 config.php 文件

```



# 已经提交但是本地在修改需要忽略的

值得注意的是中央仓库已经存在该文件后 需要取消追踪
对某个文件取消跟踪

```
git rm --cached  readme1.txt # 删除readme1.txt的跟踪，并保留在本地。

git add -A # 这里是将所有的改动提交到git

git commit -m 'update local file'
```

然后git commit 即可。但是git status查看状态时还是会列出来








