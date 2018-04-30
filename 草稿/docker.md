docker
===


> docker 是一种容器化技术，也就是软件打包技术

docker 流行的原因：

> docker 可以粗糙地理解为轻量级虚拟机

### docker 常用指令
docker -v  查看版本
docker images 查看本地 镜像
docker rmi 容器id  删除某个镜像 (rmi  remove images)

docker ps 查看docker运行进程	
docker ps -a 查看所有运行过的容器，包括已经停止的

docker run foo  运行foo 镜像 如果没有回去远程仓库里面寻找并下载这个镜像

docker run -p 8080:80 -d daocloud.io/nginx 
-p 端口映射 80端口映射本地的8080 端口
-d 后台守护进程运行
此时会返回容器ID 
docker stop  容器id 停止运行这个容器

docker cp index.html  容器id://usr/share/nginx/html
将index.html 拷贝到该容器的目录里
docker 在容器内的改变都是暂存的，一旦该容器停止这些改动就会消失。

docker commit -m 'xxx'  容器id
保持容器id的改动，该指令会返回一个哈希值， 实际上这个是新生成的容器的id值，也就是说保存一个容器的改动会生成一个新的容器。所以 一般指令 应该在后面加上新的容器名
docker commit 'xxx' 当前容器id  新生成的容器镜像名称。但是commit 主要用来入侵后保护现场，不要使用docker commit 来定制镜像，定制镜像应该使用dockerfile 来完成

命令小结
- docker pull 获取image
- docker build  创建 image
- docker images 查看本地image
- docker run 运行容器
- docker ps 列出运行的容器

