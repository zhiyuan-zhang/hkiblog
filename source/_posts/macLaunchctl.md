---
title: launchctl自动化脚本
date: 2019/4/18
categories:
  - linux
tags:
  - 自动化脚本
comments: true
abbrlink: 49197
img:
---


启动
```
launchctl start con.hki.zhxtgit.plist
```

查找
```shell
launchctl list | grep 'con'
```

加载
```shell
launchctl load -w con.hki.zhxtgit.plist
```

卸载
```shell
launchctl unload con.hki.zhxtgit.plist
```

校验语法
```shell
plutil -lint  com.hki.zhxtgit.plist
```


plist文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Disabled</key>
    <false/>
    <key>KeepAlive</key>
    <false/>
    <key>Label</key>
    <string>con.hki.zhxtgit.plist</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Documents/shell/zhxtgit.sh</string>
    </array>
    <key>RunAtLoad</key>
    <false/>
    <key>StandardErrorPath</key>
    <string>/Documents/shell/run.log</string>
    <key>StandardOutPath</key>
    <string>/Documents/shell/run.err</string>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>15</integer>
        <key>Minute</key>
        <integer>5</integer>
    </dict>
</dict>
</plist>
```
