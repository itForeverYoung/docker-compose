# docker-compose.yml

## 本文档，可以非常简单的搭建起基础开发环境

* 首先，安装docker，运行`docker -version`查看是否安装成功。

* 若安装失败，请参考**[安装docker]( https://docs.docker.com/install/ )**进行安装，

* 然后，安装docker-compose，如果是Mac和Windows的desktop版本，默认已经包含，运行`docker-compose --version`查看是否安装成功。

* 若安装失败，请参考**[安装docker-compose](https://docs.docker.com/compose/install)**进行安装。

* 新建一个yml文件，将以下内容拷贝进去，然后，在当前目录下执行`docker-compose up -d`即可。

  **v1.0**

```yaml
version: '3'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql-5.7
    hostname: mysql-5.7
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: 123456
  redis:
    image: redis
    container_name: redis
    hostname: redis
    ports:
      - "6379:6379"
  mongo:
    image: mongo
    container_name: mongodb
    hostname: mongo
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: 123456
  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    hostname: rabbitmq
    ports:
      - "5671:5671"
      - "5672:5672"
      - "15671:15671"
      - "15672:15672"
      - "4369:4369"
    environment:
        RABBITMQ_DEFAULT_USER: root
        RABBITMQ_DEFAULT_PASS: 123456
  elasticsearch:
    image: elasticsearch
    container_name: elasticsearch
    hostname: elasticsearch
    ports:
      - "9200:9200"
      - "9300:9300"
  kibana: 
    image: kibana
    container_name: kibana
    hostname: kibana
    ports:
      - "5601:5601"
    links: [elasticsearch]
```

* MySQL默认端口：3306，密码：123456。
* Redis默认端口：6379，无密码。
* ~~MongoDB默认无密码，启动成功，但外部无法连接。~~
* Rabbitmq访问地址： http://127.0.0.1:15672/，默认账户、密码：guest。
* ElasticSearch访问地址： http://127.0.0.1:9200/ ，无密码。
* kibana访问地址： http://127.0.0.1:5601/ ， kibana是ElasticSearch官方推荐的可视化管理页面。

## 下面，是升级版，添加了volumes映射，networks管理

**v1.1**

```yaml
version: '3'
services:
  mysql:
    # 镜像
    image: mysql:5.7
    # 容器名称
    container_name: mysql-5.7
    hostname: mysql-5.7
    # 开放端口
    ports:
      - "3306:3306"
    # 设置容器变量
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    # 挂载卷
    volumes:
      - "./mysql/single/conf:/etc/mysql/conf.d"
      - "./mysql/single/data:/var/lib/mysql"
      - "./mysql/single/logs:/var/log/mysql"
    # 启动时执行的命令，设置字符集，设置编码，设置时区
    command: ['mysqld', '--character-set-server=utf8mb4', '--collation-server=utf8mb4_unicode_ci', '--explicit_defaults_for_timestamp=false']
    # 是否自动重启
    restart: always
    # 加入网络组
    networks:
      - db
  redis:
    image: redis
    container_name: redis
    hostname: redis
    ports:
      - "6379:6379"
    volumes:
      - "./redis/single/conf:/etc/redis"
      - "./redis/single/data:/data"
    # 开启持久化，现已通过配置文件进行开启
    # command: ['redis-server', '--appendonly', 'yes']
    networks:
      - db
  mongo:
    image: mongo
    container_name: mongodb
    hostname: mongo
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: 123456
    # volumes:
      # - "./mongo/single/data:/data/db"
    networks:
      - db
  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    hostname: rabbitmq
    ports:
      - "5671:5671"
      - "5672:5672"
      - "15671:15671"
      - "15672:15672"
      - "4369:4369"
    environment:
        RABBITMQ_DEFAULT_USER: root
        RABBITMQ_DEFAULT_PASS: 123456
    # volumes:
      # - "${LOCAL_PATH}/rabbitmq/single/conf/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf"
      # - "${LOCAL_PATH}/rabbitmq/single/data:/var/lib/rabbitmq"
  elasticsearch:
    image: elasticsearch
    container_name: elasticsearch
    hostname: elasticsearch
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - "./elasticsearch/single/config/es-single.yml:/usr/share/elasticsearch/config/elasticsearch.yml"
      - "./elasticsearch/single/data:/usr/share/elasticsearch/data"
    # networks:
      # - db
  kibana: 
    image: kibana
    container_name: kibana
    hostname: kibana
    ports:
      - "5601:5601"
    links: [elasticsearch]
    depends_on:
      - elasticsearch
    # networks:
      # - db
networks:
  db:
```

* 基本信息和**v1.0**一致，添加了`volumes`卷映射，添加了容器间`networks`共享网络。
* 将MySQL的`data`映射到了当前目录下的`mysql/single/data`，`log`映射到了`mysql/single/logs`，配置文件映射到了`mysql/single/conf/mt.cnf`。
* 将Redis的`data`，映射到了当前目录下的`redis/single/data`，配置文件映射到了`/redis/single/conf/redis.conf`。
* 将ElasticSearch的`data`映射到了当前目录下的`elasticsearch/single/data`，配置文件映射到了`elasticsearch/single/config/es-single.yml`。
* mongo和rabbitmq的卷挂载有些问题，暂未解决。

## 对比

* **v1.0**的优点是，无需创建文件夹，只需启动即可；缺点是，数据没有保存，删除容器后，数据全部丢失了。
* **v1.1**的优点是，数据全都保存在物理磁盘中了。