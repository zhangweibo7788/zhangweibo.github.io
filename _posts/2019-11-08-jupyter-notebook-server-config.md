---
layout:     post
title: "服务器上配置Jupyter NoteBook被远程访问"
subtitle:   ""
date:       2019-11-08 10:45:00
author:     "LeeZC"
header-img: "img/article-header.png"
catalog: true
tags:
 - Jupyter NoteBook
---
> Jupyter Notebook（此前被称为 IPython notebook）是一个交互式笔记本，支持运行 40 多种编程语言。
> Jupyter Notebook 的本质是一个 Web 应用程序，便于创建和共享文学化程序文档，支持实时代码，数学方程，可视化和 markdown。 用途包括：数据清理和转换，数值模拟，统计建模，机器学习等等
> 

## 安装

Jupyter Note依赖的第三包比较多，所以不推荐大家使用系统环境的Python来安装Jupyter，建议大家虚拟一个纯净的虚拟环境来安装Jupyter，我使用的是Virtualenv+Virtualenvwrapper来控制Python第三包环境，大家记得注意Pip安装Jupyter对应的Python解释器版本，这决定安装后Jupyter的使用所依赖的Python解释器版本，这里我使用的是Pyenv来控制Python解释器版本，大家也可以使用官方推荐的Pipenv，这个根据个人的熟悉度而言。具体安装过程如下：

### 创建虚拟环境

`mkvirtualenv jupyter_workspace`

### 安装jupyter

这里我使用了国内Pip源，下载速度比较快！推荐大家使用pip管理工具安装库文件的时候使用国内Pip源。

`pip install -i https://pypi.douban.com/simple/ jupyter`

## 环境配置

### 生成默认配置文件

默认情况下，配置文件 **~/.jupyter/jupyter_notebook_config.py **并不存在，需要自行创建。使用下列命令生成配置文件：

`jupyter notebook --generate-config` 

执行成功后，会出现下面的信息：

Writing default config to: /home/lee/.jupyter/jupyter_notebook_config.py

### 生成密码

#### 自动生成
从jupyter notebook 5.0 版本开始，提供了一个命令来设置密码:

`jupyter-notebook password`

执行成功后，会出现下面的信息：

Wrote hashed password to /home/lee/.jupyter/jupyter_notebook_config.json

生成的密码存储在 **jupyter_notebook_config.json**

#### 手动生成

除了使用提供的命令自动生成，也可以通过手动安装。手动安装不方便的地方在于我们需要自己生成`sha1`，然后复制粘贴到配置文件`.jupyter/jupyter_notebook_config.py`进行配置，而且一旦我们需要修改密码的话还得重新进行一样的手动生成配置过程，这样显得很不方便。执行过程如下：

```python
from notebook.auth import passwd 
passwd()

```

![jupyter passwd](https://cdn.leezc.cn/article/jupyter-passwd.png)

### 修改配置文件

要在服务器上配置Jupyter NoteBook被远程访问，需要我们进行一些定制化配置，在 jupyter_notebook_config.py中找到下面的行，取消注释并修改。

```python
 
c.NotebookApp.ip='*'   # “*”代表非本机都可以访问

c.NotebookApp.certfile = u'/absolute/path/to/your/certificate/fullchain.pem'  # 配置SSL证书公钥的绝对路径

c.NotebookApp.keyfile = u'/absolute/path/to/your/certificate/privkey.key'  # 配置SSL证书私钥的绝对路径

c.NotebookApp.open_browser = False    # 修改为在启动notebook的时候不启动浏览器，服务器上我们不需要。

c.NotebookApp.notebook_dir = '/home/lee/jupyter_workspace'    # 指定notebook服务的目录（缺省为运行jupyter命令时用户所在的目录，注意此目录不能为隐藏目录，记得要先创建这个目录，不然会报错）

c.NotebookApp.port =9999 # 指定notebook的服务端口号，默认8888，可自行指定一个端口, 访问时使用该端口

```

## 运行jupyter

### 终端运行

`jupyter notebook`

### 后台运行

Systemd是目前新版的linux比较常用的管理后台服务的机制。在Linux的发行版Fedora、ArchLinux，Debian（8或以上），Ubuntu（15.04以上），CentOS，Redhat都使用systemd机制。这里我们使用systemd机制来配置Jupyter NoteBook来随系统自启，配置过程如下：

创建文件/etc/systemd/system/jupyter.service:

`sudo vim /etc/systemd/system/jupyter.service`

文件内容为：

```shell
[Unit]
Description=Jupyter Notebook

[Service]
Type=simple
ExecStart=/home/zclee/workspace/jupyter_workspace/bin/jupyter-notebook
User=zclee
WorkingDirectory=/home/zclee/jupyter_workspace
Group=www-data

[Install]
WantedBy=multi-user.target


```

- **ExecStart**内容为** jupyter-notebook**可执行文件所在的绝对路径，这里我使用虚拟环境安装的，所以找到虚拟环境所在位置。
- **User**内容为你当前用户名
- **WorkingDirectory**内容为配置文件**c.NotebookApp.notebook_dir**项所配置内容

配置并保存之后运行如下命令：

- `sudo systemctl daemon-reload`
- `sudo systemctl enable jupyter`
- `sudo systemctl start jupyter`

这样我们可以愉快地使用Jupyter NoteBook啦！

相关命令如下：

```shell
sudo systemctl daemon-reload  # 重新加载配置

sudo systemctl enable jupyter  # 设置系统自启动

sudo systemctl start jupyter  # 手动启动服务

sudo systemctl status jupyter  # 查看服务状态

sudo systemctl stop jupyter  # 手动停止服务

sudo systemctl restart jupyter  # 手动重启服务

sudo journalctl -f -u jupyter  # 查看日志输出

sudo journalctl -f -u jupyter | grep -i 'error'  # 查看日志输出中的error部分

sudo systemctl disable jupyter  # 自启动中去除服务


```
#### nohup后台运行

`(nohup jupyter notebook > jupyter.log 2>&1 &) `

相关命令：
```shell
lsof -i: 8899 # 查看运行端口

netstat -an | grep 8899  # 查看运行端口

tail -f/cat/more jy.log  # 查看运行日志

kill -9 pid   # 杀死后台运行进程

```
更多配置可以查阅官方文档：[Jupyter NoteBook](https://jupyter-notebook.readthedocs.io/en/latest/public_server.html#notebook-server-security)

