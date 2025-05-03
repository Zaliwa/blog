---
title: "学习安装docker并制作mininet镜像"
date: 2025-04-28
---

***很遗憾，教程出现了问题，请等待后续更新***

## 1. 安装docker
```bash
sudo apt-get update
sudo apt-get install docker.io
```

## 2. 启动docker服务
```bash
sudo systemctl start docker
sudo systemctl status docker # 查看docker服务状态
sudo systemctl enable docker # 开机启动docker服务
```

## 3. 添加国内镜像源（可选）
### 修改json文件
修改/etc/docker/daemon.json     
中间的那么多用引号括起来的网址就是镜像源，可以自行添加。
```json
{
    "registry-mirrors": [
        "https://docker.xuanyuan.me",
        "https://dockerproxy.com",
        "https://mirror.baidubce.com",
        "https://docker.m.daocloud.io",
        "https://docker.nju.edu.cn",
        "https://docker.mirrors.sjtug.sjtu.edu.cn"
    ]
}
```
这个网址里面有持续更新的最新加速源，有的镜像源没有社区镜像，加速源可以下载社区镜像。链接：https://cloud.tencent.com/developer/article/2485043
### 加载已添加的镜像源
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```
### 检查是否添加成功
```bash
sudo docker info
```
如果看到registry-mirrors后面有网址，说明添加成功。

## 4. 拉取ubuntu镜像
先试试hello world。这里的run会自动拉取镜像并启动容器。
```bash
sudo docker run hello-world
```
没有魔法的话会失败，请看下面的4. 国内镜像源
解决问题后，拉取ubuntu镜像
```bash
sudo docker pull ubuntu:22.04
```
查看镜像
```bash
sudo docker images
```

## 5. 制作mininet镜像
从ubuntu镜像打开容器，进入容器后，安装mininet
```bash
sudo docker run -it ubuntu:22.04
```
安装mininet
```bash
apt-get update
apt-get install mininet
```
安装完成后，退出容器
```bash
exit
```
检查容器是否关闭，以及查看容器ID
```bash
sudo docker ps -a
```
将容器保存为镜像
```bash
sudo docker commit <container_id> mininet-ubuntu:22.04  # 这里的mininet-ubuntu:22.04是镜像的名称，可以自行修改，id可以只写前几个字符
```
查看镜像，可以看到mininet-ubuntu:22.04已经存在了
```bash
sudo docker images
```
删除容器
```bash
sudo docker rm <container_id>
```

## 6. 添加用户到docker组（可选）
```bash
sudo usermod -aG docker $USER
```
注销后重新登录，或者执行以下命令使更改生效
```bash
newgrp docker
```
查看用户是否加入docker组
```bash
docker info
```
也就是说，你现在可以使用docker命令了，不需要sudo。

## 7. 首次运行mininet镜像
```bash
docker run -it --name mininet-ubuntu -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -v /home/your-workspace:/home/user -v /usr/include/X11:/usr/include/X11 --restart=no -w /home/user mininet-ubuntu:22.04
```
解释：
- -it：交互式运行
- --name mininet-ubuntu：容器名称mininet-ubuntu
- -e DISPLAY=$DISPLAY：共享X服务器
- -v -v /tmp/.X11-unix:/tmp/.X11-unix -v /usr/include/X11:/usr/include/X11：共享X服务器的配置，通过把配置和库文件挂载到容器中，使得容器可以访问X服务器
- -v /home/your-workspace:/home/user：将宿主机的/home/your-workspace目录挂载到容器的/home/user目录，这样容器就可以访问宿主机的文件了
- -w /home/user：指定容器的工作目录
- --restart=no：不重启容器，指的是不开机启动（大概）
- mininet-ubuntu:22.04：镜像名称
### 关于X服务器的共享
在宿主机上，执行以下命令
```bash
xhost +:local
```
在容器中打开mininet的GUI(miniedit)就可以看到在宿主机上的窗口。

## 8. 创建容器后的容器管理
### 查看容器
```bash
docker ps -a
```
### 启动容器
```bash
docker start mininet-ubuntu # 启动容器，容器名称为mininet-ubuntu，也可以使用容器ID
```
### 进入容器
```bash
docker exec -it mininet-ubuntu bash 
```
### 退出容器
```bash
exit
```
### 停止容器
```bash
docker stop mininet-ubuntu # 停止容器，容器名称为mininet-ubuntu，也可以使用容器ID
```
### 删除容器（等你不再需要它）
```bash
docker rm mininet-ubuntu # 删除容器，容器名称为mininet-ubuntu，也可以使用容器ID
```
### 删除镜像（等你不再需要它）
```bash
docker rmi mininet-ubuntu:22.04 # 删除镜像，镜像名称为mininet-ubuntu:22.04，也可以使用镜像ID
```