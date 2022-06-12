[TOC]

（待补充）

# systemd

Unit

每一个 Unit（服务等） 都有一个配置文件，告诉 Systemd 怎么启动这个 Unit 。

Systemd 默认从目录 /etc/systemd/system/ 读取配置文件。

systemctl reload nginx 重新加载 nginx 配置文件

systemctl restart nginx 重新启动 nginx 服务

## Service

service 文件怎么写，待补充

## Unit

待补充

## journal

CentOS 7 以后版本，利用Systemd 统一管理所有 Unit 的**启动**日志。带来的好处就是，可以只用 journalctl 一个命令，查看所有日志（内核日志和应用日志）。

systemd为我们提供了一个统一中心化的日志系统: journal

其中包含了守护线程 journald 以及我们用来查看日志的工具 journalctl 等等。

journald 任劳任怨，来者不拒地收集来自各个应用和内核的日志信息。

### journal 命令

```bash
# 显示尾部的最新 10 行日志
journalctl -n
# 显示尾部指定行数的日志
journalctl -n 20
# 查看某个 Unit 的日志
journalctl -u nginx.service
# -r reverse 从尾部看日志
journalctl -r
# journalctl 日志太长了会被截断显示不全
journalctl -n 40 -u nginx.service 
# 建议使用 vim 打开查看日志
journalctl -n 40 -u nginx.service | vim -
```

