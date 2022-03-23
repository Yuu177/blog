[TOC]

# ansible

目标：ansible 配置，代码仓部署到测试环境。https://confluence.shopee.io/pages/viewpage.action?pageId=897349013

## ansible 介绍

官方介绍：ansible 是新出现的自动化运维工具，基于 Python 开发，集合了众多运维工具（puppet、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能。

举个例子：我们可以理解为是一个工具，可以批量处理多台服务器的机器。举个例子，如果我要在2 台服务器都安装一个 nginx，然后配置相关代理，上传对应的静态资源。那么我们常规的操作，就是先 ssh 登录一台机子，然后手动处理完这些任务后，再登录一台机子，继续手动相关操作。通过 ansible，它会帮我们批量的操作 2 台机子。底层的原理：通过配置好的 sshkey，可以连接到配置好 host 的机器（这两台机子），然后通过相关命令，批量操作 2 台机子。

## Ansible的幂等性

介绍 ansible 特点时说过，ansible 具有幂等性，幂等性能够保证我们重复的执行一项操作时，得到的结果是相同的，下面详细介绍一下幂等性的概念。

举个例子，你想把一个文件拷贝到目标主机的某个目录上，但是你不确定此目录中是否已经存在此文件，当你使用 ansible 完成这项任务时，就非常简单了，因为如果目标主机的对应目录中已经存在此文件，那么 ansible 则不会进行任何操作，如果目标主机的对应目录中并不存在此文件，ansible 就会将文件拷贝到对应目录中。说白了，ansible 是”以结果为导向的”，我们指定了一个”目标状态”，ansible 会自动判断，”当前状态”是否与”目标状态”一致，如果一致，则不进行任何操作，如果不一致，那么就将”当前状态”变成”目标状态”，这就是”幂等性”，”幂等性”可以保证我们重复的执行同一项操作时，得到的结果是一样的。

这种特性在很多场景中对于脚本来说都有一定的优势。

## ansible 任务执行

Ansible 系统由控制主机对被管节点的操作方式可分为两类，即 ad-hoc 和 playbook。

- ad-hoc 模式(点对点模式)

  使用单个模块，支持批量执行单条命令。ad-hoc 命令是一种可以快速输入的命令，而且不需要保存起来的命令。**就相当于在 shell 中输入一行命令。**

  - 如 `ansible 172.25.63.5 -m ping`，ansible 后面接的是 hosts，-m 后面是模块名。

- playbook 模式(剧本模式)

  是 Ansible 主要管理方式，也是 Ansible 功能强大的关键所在。**playbook 通过多个 task 集合完成一类功能**，如 Web 服务的安装部署、数据库服务器的批量备份等。可以简单地把 playbook 理解为通过组合多条 ad-hoc 操作的配置文件。**就相当于 shell 脚步。**

## ansible playbook

在上面的流程中，我们可以通过 ansible，通过对应的命令，批量操作 2 台机器，但是这样也是要手动执行对应命令的，每次都要输入 ansible 的命令，很麻烦。为了实现全自动，衍生出了ansible playbook（剧本）。我们可以通过编写对应的 yml，让 ansible 按照我们编写好的剧本去执行对应的操作。

## 模块命令详解

```yaml
- name: Deploy test Server
  tasks:
    - name: include ip addr
      include_vars:
        file: "{{config_ip_addr}}" # 这里意思是包含 config_ip_addr 这个文件里的所有变量
```

Ansible 默认只会对控制机器执行操作，但如果在这个过程中需要在 Ansible 本机执行操作呢？细心的读者可能已经想到了，可以使用 delegate_to( 任务委派 ) 功能呀。没错，是可以使用任务委派功能实现。不过除了任务委派之外，还可以使用另外一外功能实现，这就是 local_action 关键字。

### command模块

模块介绍
command模块可以帮助我们在远程主机上执行命令

注意：使用command模块在远程主机中执行命令时，不会经过远程主机的shell处理，在使用command模块时，如果需要执行的命令中含有重定向、管道符等操作时，这些符号也会失效，比如”<“, “>”, “|”, “;” 和 “&” 这些符号，如果你需要这些功能，可以参考后面介绍的shell模块，还有一点需要注意，如果远程节点是windows操作系统，则需要使用win_command模块。



### shell模块

#### 模块介绍

shell模块可以帮助我们在远程主机上执行命令，**与command模块不同的是，shell模块在远程主机中执行命令时，会经过远程主机上的/bin/sh程序处理。**



ansible中关于模块的命令

列出ansible所支持的模块：

`ansible-doc -l`

查看模块的详细帮助信息，比如fetch：

`ansible-doc -s fetch`

调用模块，比如调用ping模块：

`ansible all -m ping`

调用模块的同时传入相关参数，以fetch为例：

`ansible testA -m fetch -a "src=/etc/fstab dest=/testdir/ansible"`

ansible 后面紧跟的是 hosts。



## ansible playbook中handlers的用法

关键点，task 中通过关键字 notify 来调用 handler

https://blog.csdn.net/qq_35887546/article/details/105121336



## tags

**tags可以帮助我们对任务进行’打标签’的操作**，当任务存在标签以后，我们就可以在执行playbook时，借助标签，指定执行哪些任务，或者指定不执行哪些任务了

https://blog.csdn.net/qq_35887546/article/details/105122999



变量

```handlebars
"{{httpd.conf80}}"
```



## register

**ansible的模块在运行之后，其实都会返回一些”返回值”**，只是默认情况下，这些”返回值”并不会显示而已，我们可以把这些返回值写入到某个变量中，这样我们就能够通过引用对应的变量从而获取到这些返回值了，这种**将模块的返回值写入到变量中的方法被称为”注册变量”**

通过 register 将返回值注册到变量中。

https://blog.csdn.net/qq_35887546/article/details/105147929