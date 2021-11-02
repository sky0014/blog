# 在Ubuntu上安装Docker和Docker Compose最简单的方法

**以下均为root账号操作，非root用户请自行添加sudo**

如果服务器在国内，先更换国内源，安装更快

```bash
sed -i s@/archive.ubuntu.com/@/mirrors.163.com/@g /etc/apt/sources.list
sed -i s@/security.ubuntu.com/@/mirrors.163.com/@g /etc/apt/sources.list
apt-get update
```

Ubuntu源中已存在Docker和Docker Compose，直接安装即可，网上一些文章中写的脚本安装或执行一些命令进行安装已经过时。

```bash
apt-get install docker docker-compose -y
```

安装好之后，可以再更换下Docker源

```
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://hub-mirror.c.163.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```

上面用的是网易源，如果你有阿里云账户，也可以在这里找到阿里云的加速地址：[容器镜像服务/镜像加速器](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)
