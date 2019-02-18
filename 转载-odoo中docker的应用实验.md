## 转载--odoo中docker的应用实验

转自 pengyb 笔记  原文链接 http://60af80da.wiz03.com/share/s/1wHU3q1f5khe2BECg11yoH_v0XtdCX1s1k0G2lUz_X0wMJHZ
准备

服务器操作系统 ubuntu 存放地点：阿里云 开发的模块存放目录 /home/tianxing_addons

安装docker
sudo apt-get install docker

查看docker版本
docker version

修改国内镜像地址
参见 https://yq.aliyun.com/articles/29941

查看网络库中与odoo有关的镜像
docker search odoo

下载镜像
odoohost/odoo 为一个包括odoo10,postgresql等的完整运行环境.
docker pull odoohost/odoo

根据镜像 创建一个运行odoo容器 并命名为 police
–name 指定容器名称 police是容器名称
-i 交互模式运行，可以ssh连上 ssh 连上的指令是 docker exec -i -t police /bin/bash
-d 后台运行
-p 映射一个容器的端口到主机的端口 9069主机端口 8069 容器端口
-v 映射一个主机目录到容器的目录 /home/tianxing_addons是主机目录 /extra-addons是容器的第三方模块目录
odoohost/odoo 是镜像
docker run –name police -i -d -p 9069:8069 -v /home/tianxing_addons:/extra-addons -e PYTHONPATH='/extra-addons' odoohost/odoo

成功以后，那可以用 http://xxx:9069
 来访问了 其实xx为服务器的域名或ip

查看正在运行的容器的日志
docker logs -f police

查看正在运行的容器
docker ps

查看正在运行的容器的端口
docker port police

