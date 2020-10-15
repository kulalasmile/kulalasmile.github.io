[TOC]

# Docker #

> 安装

```shell
#安装依赖包
yum install -y yum-utils device-mapper-persistent-data lvm2
#添加Docker软件包源
yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
#安装Docker CE
yum install docker-ce -y
#启动
systemctl start docker
#开机启动
systemctl enable docker
#查看Docker信息
docker info
```

![](https://gitee.com//kulalasmile/image/raw/master/img/20200801095011.png)

```
vim /etc/docker/daemon.json

{
  "registry-mirrors": ["https://b4lxht7n.mirror.aliyuncs.com"]
}
```


https://b4lxht7n.mirror.aliyuncs.com


> 卸载

1. 查看已安装的docker列表

   ```shell
   yum list installed | grep docker
   ```

2. 卸载docker

   ```shell
   yum remove docker \
                      docker-client \
                      docker-client-latest \
                      docker-common \
                      docker-latest \
                      docker-latest-logrotate \
                      docker-logrotate \
                      docker-selinux \
                      docker-engine-selinux \
                      docker-engine
                      
   rm -rf /etc/systemd/system/docker.service.d
   
   rm -rf /var/lib/docker
    
   rm -rf /var/run/docker
   ```

   

> 命令

- 启动docker服务

  - ```shell
    sudo service docker start
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520102056.png)

- 镜像相关

  提示：**对于镜像的操作可使用镜像名、镜像长ID和短ID**。 


  - 查看本地的镜像

    ```dockerfile
    docker images
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520143212.png)

  - 搜索镜像

    ```dockerfile
    docker search " "
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520154320.png)

  - 下载镜像

    ```dockerfile
    ##下载Redis官方最新镜像，相当于：docker pull redis:latest
    docker pull redis
    ##指定版本
    dockre pull redis:3.2
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520154412.png)

  - 单个镜像删除

    ```dockerfile
    docker rmi "IMAGE ID"
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520142519.png)

    



- 容器操作

  提示：**对于容器的操作可使用CONTAINER ID 或 NAMES**。 

  - 运行容器

    ```dockerfile
    docker run --name redis -p 16379:16379 -d redis --requirepass "123456"
    #常用选项说明
    ## -d, --detach=false， 指定容器运行于前台还是后台，默认为false
    ## -u, --user=""， 指定容器的用户
    ## -w, --workdir=""， 指定容器的工作目录
    ## -P, --publish-all=false， 指定容器暴露的端口
    ## --name=""， 指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字
    ## -v,挂载宿主机的一个目录
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo1589957998395.png)

  - 容器启动

    ```dockerfile
    docker start redis
    ## 第一次运行后，第二次可以使用start 不用带参数
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520150852.png)

  - 查看正在运行的容器 

    ```
    docker ps
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520151210.png)

  - 查看正在运行+历史运行的容器

    ```
    docker ps -a
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520151440.png)

  - 显示运行容器总文件大小 

    ```
    docker ps -s
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520151552.png)

  - 容器重启

    ```
    docker restart redis
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520151003.png)

  - 容器停止

    ```
    docker stop redis
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520150605.png)

  - 容器日志查看

    ```
    docker logs redis
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520151927.png)

  - 列出容器中运行进程 

    ```
    docker top redis
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520151758.png)

  - 查看正在运行+历史运行过的容器 

    ```dockerfile
    docker ps -a
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520103931.png)


  - 删除容器

    ```dockerfile
    docker rm "NAMES"
    ```

    ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520154603.png)



> 目录挂载

 如果要挂载宿主机的一个目录，可以用-v参数指定。  譬如我要启动一个centos容器，宿主机的/test目录挂载到容器的/soft目录 ， 冒号":"前面的目录是宿主机目录，后面的目录是容器内目录 

```shell
[root@localhost ~]# docker run -it -v /test:/soft centos /bin/bash
[root@a487a3ca7997 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  soft  srv  sys  tmp  usr  var
```

现在镜像内就可以共享宿主机里的文件了。

默认挂载的路径权限为读写。如果指定为只读可以用：ro

```
[root@localhost ~]# docker run -it -v /test:/soft:ro centos /bin/bash
```







> Dockerfile

[案例](https://www.cnblogs.com/edisonchou/p/dockerfile_inside_introduction.html)


> 进入容器命令

```
docker exec -it redis /bin/bash
```


> 查看实时日志

```
docker logs -f -t --since="2020-10-10" --tail=10 redis
```

--since : 此参数指定了输出日志开始日期，即只输出指定日期之后的日志。

-f : 查看实时日志

-t : 查看日志产生的日期

-tail=10 : 查看最后的10条日志。

edu_web_1 : 容器名称

> 安装nacos

```
docker run -d -e PREFER_HOST_MODE=ip -e MODE=standalone -e SPRING_DATASOURCE_PLATFORM=mysql -e MYSQL_SERVICE_HOST=172.17.0.2 -e MYSQL_SERVICE_PORT=3306 -e MYSQL_SERVICE_USER=root -e MYSQL_SERVICE_PASSWORD=yf987654321 -e MYSQL_SERVICE_DB_NAME=nacos -e JVM_XMS=500m -e JVM_XMX=500m -e JVM_XMN=400m -v /usr/local/nacos/logs:/home/nacos/logs -p 8848:8848 --name nacos  nacos/nacos-server:1.3.2
```

> 安装mysql

```
docker run -p 3306:3306 --name mysql5.7 -v /usr/local/mysql57/conf:/etc/mysql/conf.d -v /usr/local/mysql57/logs:/logs -v /usr/local/mysql57/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=yf987654321 -d mysql:5.7
```

> 安装minio

```
docker run --name minio -d -p 9000:9000 -e MINIO_ACCESS_KEY=minio -e MINIO_SECRET_KEY=yf987654321 -v /usr/local/minio/data:/data  minio/minio server /data
```

> 安装rabbitmq

```
docker run -d --hostname rabbitmq --name rabbitmq -e RABBITMQ_DEFAULT_USER=rabbitmq -e RABBITMQ_DEFAULT_PASS=yf987654321 -p 15672:15672 -p 5672:5672 rabbitmq:management
```

> 安装sentinel

```
docker run --name sentinel -d -p 8858:8858 -d bladex/sentinel-dashboard
```

> 安装elasticsearch

```
 docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx128m" -v /usr/local/elasticsearch/data:/usr/share/elasticsearch/data -v /usr/local/elasticsearch/plugins:/usr/share/elasticsearch/plugins -d elasticsearch:7.9.2
```

> 安装kibana 

```
docker run --name kibana -p 5601:5601 \
--link elasticsearch:es \
-e "elasticsearch.hosts=http://es:9200" \
-e "I18N_LOCALE=zh-CN"\
-d kibana:7.9.2
```

> 安装redis

```
docker run -p 16379:6379 --name redis \
-v /usr/local/redis/data:/data \
-d redis:5.0.9 redis-server --appendonly yes --requirepass "yf987654321"
```

> 安装mongo

```
docker run -p 27017:27017 --name mongo \
-v /usr/local/mongo/db:/data/db \
-d mongo:4.2.10 --auth

```

```
docker exec -it mongo mongo
```

```
use admin
db.createUser({ 
    user: 'mongoadmin', 
    pwd: 'yf987654321', 
    roles: [ { role: "root", db: "admin" } ] });
```

