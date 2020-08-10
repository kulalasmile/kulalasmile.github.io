
 [TOC]

# Docker

## 命令

- 启动docker服务

  ```shell
  sudo service docker start
  ```

  ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520102056.png)

- 镜像相关

   提示：**对于镜像的操作可使用镜像名、镜像长ID和短ID**。 
   
   
  -  查看本地的镜像
  
     ```dockerfile
     docker images
     ```
  
     ![](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGo20200520143212.png)
  
  -  搜索镜像
  
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

  -  显示运行容器总文件大小 

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

  -  列出容器中运行进程 

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



## 目录挂载

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



## Dockerfile

[案例](https://www.cnblogs.com/edisonchou/p/dockerfile_inside_introduction.html)



