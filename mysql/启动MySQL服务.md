# 启动MySQL服务

### 本文档，可以简单的搭建起MySQL单机版或集群

* 首先，安装docker，运行`docker -version`查看是否安装成功。
* 若安装失败，请参考**[安装docker]( https://docs.docker.com/install/ )**进行安装，
* 然后，安装docker-compose，如果是Mac和Windows的desktop版本，默认已经包含，运行`docker-compose --version`查看是否安装成功。
* 若安装失败，请参考**[安装docker-compose](https://docs.docker.com/compose/install)**进行安装。

## MySQL单机版搭建

* 首先，在当前目录下创建以下目录和文件。

  ```
  mysql/single/conf/my.cnf
  mysql/single/data
  mysql/single/logs
  ```

* 然后，新建一个`mysql-standalone.yml`文件，将以下内容拷贝进去。

* 最后，在当前目录下执行`docker-compose -f mysql-standalone.yml up -d`，MySQL服务启动成功。

  ```sql
  version: "3"
  services:
    mysql-standalone:
        image: mysql:5.7
        container_name: mysql-standalone
        hostname: mysql-5.7
        ports:
          - "3306:3306"
        environment:
          MYSQL_ROOT_PASSWORD: 123456
        volumes:
          - "./single/data:/var/lib/mysql"
          - "./single/conf:/etc/mysql/conf.d"
          - "./single/logs:/var/log/mysql"
        # restart: always
        networks:
          - db
  networks:
    db:
  ```

***

## MySQL主从复制搭建

### 下面开始启动MySQL服务

* 首先，在当前目录下创建以下目录和文件。

  ```yml
  # master
  mysql/master/conf/my.cnf
  mysql/master/data
  mysql/master/logs
  # slave-1
  mysql/slave-1/conf/my.cnf
  mysql/slave-1/data
  mysql/slave-1/logs
  # slave-2
  mysql/slave-2/conf/my.cnf
  mysql/slave-2/data
  mysql/slave-2/logs
  ```

* 然后，新建一个`mysql-cluster.yml`文件，将以下内容拷贝进去。

* 最后，在当前目录下执行`docker-compose -f mysql-cluster.yml up -d`，MySQL服务启动成功。


```yaml
version: "3"
services:
  mysql-master:
      image: mysql:5.7
      container_name: mysql-master
      hostname: mysql-5.7
      ports:
        - "3306:3306"
      environment:
        MYSQL_ROOT_PASSWORD: 123456
      volumes:
        - "./master-slave/master/data:/var/lib/mysql"
        - "./master-slave/master/conf:/etc/mysql/conf.d"
        - "./master-slave/master/logs:/var/log/mysql"
      # restart: always
      networks:
        - db
  mysql-slave-1:
      image: mysql:5.7
      container_name: mysql-slave-1
      hostname: mysql-5.7
      ports:
        - "3316:3306"
      environment:
        MYSQL_ROOT_PASSWORD: 123456
      volumes:
        - "./master-slave/slave-1/data:/var/lib/mysql"
        - "./master-slave/slave-1/conf:/etc/mysql/conf.d"
        - "./master-slave/slave-1/logs:/var/log/mysql"
      # restart: always
      networks:
        - db
  mysql-slave-2:
      image: mysql:5.7
      container_name: mysql-slave-2
      hostname: mysql-5.7
      ports:
        - "3326:3306"
      environment:
        MYSQL_ROOT_PASSWORD: 123456
      volumes:
        - "./master-slave/slave-2/data:/var/lib/mysql"
        - "./master-slave/slave-2/conf:/etc/mysql/conf.d"
        - "./master-slave/slave-2/logs:/var/log/mysql"
      # restart: always
      networks:
        - db
networks:
  db:
```

### 接下来，进行主从配置

```sql
# 登录mysql
mysql -uroot -p123456
# 进入master容器
docker exec -it mysql-master bash
# 创建一个用户，供同步使用
grant all privileges on *.* to 'slave'@'%' identified by "slave_123456";
# 刷新权限
flush privileges;
# 查看master同步状态，获取master_log_file和master_log_pos的值
show master status \G;

# 分别进入slave容器
# 关闭slave同步进程
stop slave;
# 连接master，master_host是master的ip，master_user和master_password是在master中创建的用户，master_log_file和master_log_pos的值为从master中获取的。
change master to master_host='mysql-master', master_port=3306, master_user='slave', master_password='slave_123456', master_log_file='mysql-bin.000010', master_log_pos=1243;
# 启动slave同步进程
start slave;
# 查看slave同步状态
show slave status \G;
# 当slave_io_running和slave_sql_running都为yes，代表配置成功
```

