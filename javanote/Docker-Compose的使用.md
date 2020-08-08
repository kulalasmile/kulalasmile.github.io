# Docker-Compose的使用

## 介绍

Docker Compose是一个用于定义和运行多个docker容器应用的工具。使用Compose你可以用YAML文件来配置你的应用服务，然后使用一个命令，你就可以部署你配置的所有服务了。



## 安装

- 下载docker-compose

  ```shell
  curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
  ```

- 修改权限为可执行

  ```shell
  chmod +x /usr/local/bin/docker-compose
  ```

- 查看是否按照成功

  ```shell
  docker-compose --version
  ```

  

































## 附录

安装springcloud alibaba环境

```dockerfile
version: "3"
services:
  mysql:
    container_name: mysql
    image: mysql:5.7
    environment:
      - MYSQL_ROOT_PASSWORD=123456
    volumes:
      - /app/cloud/mysql/data:/var/lib/mysql
    ports:
      - "3306:3306"
    restart: always

  nacos:
    image: nacos/nacos-server:1.1.4
    container_name: nacos
    environment:
      - PREFER_HOST_MODE=hostname
      - MODE=standalone
      - MYSQL_DATABASE_NUM=1
      - SPRING_DATASOURCE_PLATFORM=mysql
      - MYSQL_MASTER_SERVICE_HOST=mysql
      - MYSQL_MASTER_SERVICE_DB_NAME=nacos_config
      - MYSQL_MASTER_SERVICE_PORT=3306
      - MYSQL_MASTER_SERVICE_USER=root
      - MYSQL_MASTER_SERVICE_PASSWORD=123456
    volumes:
      - /app/cloud/nacos/logs:/home/nacos/logs
    ports:
      - "8848:8848"
    depends_on:
      - mysql
    restart: always


  sentinel:
    image: bladex/sentinel-dashboard:latest
    container_name: sentinel
    ports:
      - "8858:8858"
    restart: always

```

