---
title: linux语法操作
date: 2019/12/28
categories:
  - linux
tags:
  - maven
comments: true
abbrlink: 49197
img:
---



# >File	
```
ls       
ll     
ls -l  
ll -rt
```


# >shell base 	
```
Ctrl+w:删除光标前面的单词的字符
Ctrl – a ：移到行首
Ctrl – e ：移到行尾
esc - f  : 前移一个词
esc - b:  后移一个词
```


创建文件用touch    	
```
例如：touch [1.txt]
```

删除文件用rm        	
```
例如：rm -f [1.txt]
```

创建目录用mkdir    	
```
例如：mkdir [xxx]     { -p 参数来创建多级文件夹 }
```
删除空目录用rmdir 	
```
例如：rmdir [xxx]（有东西的目录不能删）
```

打开文件	

```
vi  vim open cat more less
```
批量创建	

```
mkdir -p nginxdb/{conf,conf.d,html,logs}
```


# >删除

删除装有东西的目录就用

```rm -rfi 	例如rm -rfi [XXX]
i是为了提醒 最好加上
```



# >查看
文件末尾	
```
Tail -20 [filename]
# >经常查看日志需要用到这个
tail -f xxx.log -n100
```



重新命名	
```
Mv [old.text] [new.text]
```

# >杀死进程	
```
Kill [pid]
```

pkill和阿里源码里学的
直接删除程序对应的进程
```
pkill -f zwkj-0.0.1-SNAPSHOT.jar 
```


# >权限赋予	
```
Chmod -R 777 [目录]
建议了解下 linux权限的 0124 组成 这样就知道777怎么来的
chown apple/staff xxx
```

# >解压	
```

zip all.zip *.jpg

unzip all.zip 
```

# >Grep 管道查询	
```

-A num：匹配到搜索到的行以及该行下面的num行
  
-B num：匹配到搜索到的行以及该行上面的num行

-C num：匹配到搜索到的行以及上下各num行


grep -E -B 1  'use time:' /data/home/zhxt/zhxt-test/logs/test.log

ls | grep '.docx'

ls | grep a*   (a 开头的)

ls | grep 'a*'  (包含a的)

https://www.cnblogs.com/kongzhongqijing/articles/4462793.html
```


# >查看历史输入的命令	

```
cat ~/.bash_history

history | head -20
```

# >查看执行文件的路径	

```
whereis mysql
```


# >读取文件	

```
Cat [filename]
```

# >查询文件	

```
find [path_root] -name '*.doc*'
find / -name [xxx] -d
locate 
```


# >查看某个端口是否被占用
习惯用lsof  不想用netstat
```
安装 
yum install lsof -y	
lsof -i tcp:8080     (PS 必须用root权限)

netstat -tunlp|grep [端口号]
ps -axu|grep [java]  启动位置输出
```
# >文件下载	
```
wget http://mirrors.shu.edu.cn/apache/tomcat/tomcat-8/v8.5.34/bin/apache-tomcat-8.5.34.zip --no-check-certificate
```

# >scp下载
```
scp -P 2204 isinonet@106.37.74.50:/home/gzzx/excel/ProfessionReport18584791024.xlsx /Users/apple/Documents
scp -P 2204 isinonet@106.37.74.50:/logs/test.log /Users/apple/Documents
-r 文件夹   -p端口
```

# >上传
服务器文件移动	
```
scp /Users/apple/Documents/zwsj-category-0.0.1-SNAPSHOT.jar root@192.168.0.118:/home/zwsjObj
Scp -r 
```


# >其他



查看磁盘容量	
```

df -hl /xxx 
```

查看位置	
```

which java
```

rpm包安装	
```

rpm -ivh **.rpm
```

后台运行命令	
```

Nohup 详细指南
```

该命令用来列出目前与过去登录系统的用户相关信息	
```

Last
```

Linux查看文件夹大小	
```

du -sh 
```

查看当前文件夹大小
```

du -sh * | sort -n 
```

统计当前文件夹(目录)大小，并按文件大小排序
```

du -sk filename 查看指定文件大小
```

遇到其他不会的命令实例可以按照这个命令来查看基础语法	
```
tldr  mkdir
```