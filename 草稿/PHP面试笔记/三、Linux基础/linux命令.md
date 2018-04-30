linux命令
===

### 考点
- 写出尽可能多的linux命令
- 系统定时任务
-  vi/vim 编辑器
- shell 基础

### Linux 命令

#### 系统安全类
- sudo
- su
- chmod
- chown 
- setfacl

#### 进程管理
- w
- top
- ps
- kill
- pkill
- pstree
- killall

#### 用户管理
- id
- usermod
- useradd
- groupadd
- userdel

#### 文件系统
- mount
- umount
- fsck
- df
- du

#### 关机重启
- showdown
- poweroff
- reboot

#### 网络相关
- curl
- telnet
- mail
- elinks
- nc

#### 网络测试
- ping
- netstat
- host

#### 网络配置
- hostname
- ifconfig

#### 常用工具
- ssh
- screen
- clear
- who 
- date

#### 软件包管理
- yum 
- rpm
- apt

#### 文件查找比较
- locate
- find
 
#### 文件内容查看
- touch
- unlink
- rename
- ln
- cat

#### 目录操作

### 定时任务
> crontab 是定时循环执行，at 是定时执行一次
#### crontab 任务
- crontab -e 创建任务
***** 命令 (分 时 日 月 周)
#### at 命令
#at 2:00 tomorrow

### vi

#### 模式
- 一般模式
删除，复制，粘贴
切换到编辑模式，i 、I、o、O、a 、A、r、R
- 编辑模式
- 命令模式
- 视图模式

### shell 基础

#### 脚本执行方式
- 赋予权限直接执行
chmod +x test.sh ;   ./test.sh
- 调用解释器执行
例如 bash csh ash bsh ksh 
- 使用source 命令
source test.sh

#### 编写基础
- 开头使用 #! 指定解释器路径 例如：#!/bin/sh
- 编写具体功能

### 面试题

#### 如何实现每天0点重启服务器
```
contrab -e
0 0 *** reboot
```