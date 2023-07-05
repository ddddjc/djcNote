# linux操作

## 1、开放端口

firewall-cmd --zone=public --add-port=5672/tcp --permanent   # 开放5672端口
firewall-cmd --zone=public --remove-port=5672/tcp --permanent  # 关闭5672端口
firewall-cmd --reload   # 配置立即生效

## 2、查看防火墙所有开放的端口

firewall-cmd --zone=public --list-ports

## 3、关闭防火墙

如果要开放的端口太多，嫌麻烦，可以关闭防火墙，安全性自行评估

systemctl stop firewalld.service

### 4、查看防火墙状态

 firewall-cmd --state

## 5、查看监听的端口

使用命令 nmap  ip地址

或者命令 netstat -lnpt
PS:centos7默认没有 netstat 命令，需要安装 net-tools 工具
yum install -y net-tools

## 6、检查端口被哪个进程占用

netstat -lnpt |grep 5672

## 7、中止进程

kill -9 6832 # 6832是进程id




systemctl status firewalld.service（查看防火墙状态）

active表示当前防火墙处于开启状态 inactive表示关闭状态
systemctl stop firewalld.service （关闭防火墙）
systemctl start firewalld.service （开启防火墙）
systemctl disable firewalld.service （禁止防火墙自启动）
systemctl enable firewalld.service （防火墙随系统开启启动）
