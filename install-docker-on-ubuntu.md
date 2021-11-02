# 在Ubuntu上安装Docker和Docker Compose最简单的方法

Ubuntu源中已存在Docker和Docker Compose，直接安装即可，网上一些文章中写的脚本安装或执行一些命令进行安装已经过时。

```bash
apt-get install docker docker-compose -y
```

如果服务器在国内，可更换国内源，安装更快

```bash
sed -i s@/archive.ubuntu.com/@/mirrors.163.com/@g /etc/apt/sources.list
sed -i s@/security.ubuntu.com/@/mirrors.163.com/@g /etc/apt/sources.list
apt-get update
```
