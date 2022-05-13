[TOC]

# systemd

（待补充）

每一个 Unit（服务等） 都有一个配置文件，告诉 Systemd 怎么启动这个 Unit 。
Systemd 默认从目录 /etc/systemd/system/ 读取配置文件。

systemctl reload nginx 重新加载 nginx 配置文件

systemctl restart nginx 重新启动 nginx 服务