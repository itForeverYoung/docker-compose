# MySQL主从复制搭建

## 本文档，可以简单的搭建起MySQL集群

* 首先，安装docker，运行`docker -version`查看是否安装成功。
* 若安装失败，请参考**[安装docker]( https://docs.docker.com/install/ )**进行安装，
* 然后，安装docker-compose，如果是Mac和Windows的desktop版本，默认已经包含，运行`docker-compose --version`查看是否安装成功。
* 若安装失败，请参考**[安装docker-compose](https://docs.docker.com/compose/install)**进行安装。

### 下面开始启动MySQL服务

* 首先，在当前目录下创建以下目录和文件。

  ```yml
  # master
  mysql/master/conf/my.cnf
  mysql/master/data
  mysql/master/logs
  # slaver-1
  mysql/slaver-1/conf/my.cnf
  mysql/slaver-1/data
  mysql/slaver-1/logs
  # slaver-2
  mysql/slaver-2/conf/my.cnf
  mysql/slaver-2/data
  mysql/slaver-2/logs
  ```

* 然后，新建一个`mysql-cluster.yml`文件，将以下内容拷贝进去。

* 最后，在当前目录下执行`docker-compose -f mysql-cluster.yml up -d`，MySQL服务启动成功。

* **注：LOCAL_PATH是在.env中配置的通用变量。**

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
        - "${LOCAL_PATH}/mysql/master/data:/var/lib/mysql"
        - "${LOCAL_PATH}/mysql/master/conf:/etc/mysql/conf.d"
        - "${LOCAL_PATH}/mysql/master/logs:/var/log/mysql"
      # restart: always
      networks:
        - db
  mysql-slaver-1:
      image: mysql:5.7
      container_name: mysql-slaver-1
      hostname: mysql-5.7
      ports:
        - "3316:3306"
      environment:
        MYSQL_ROOT_PASSWORD: 123456
      volumes:
        - "${LOCAL_PATH}/mysql/slaver-1/data:/var/lib/mysql"
        - "${LOCAL_PATH}/mysql/slaver-1/conf:/etc/mysql/conf.d"
        - "${LOCAL_PATH}/mysql/slaver-1/logs:/var/log/mysql"
      # restart: always
      networks:
        - db
  mysql-slaver-2:
      image: mysql:5.7
      container_name: mysql-slaver-2
      hostname: mysql-5.7
      ports:
        - "3326:3306"
      environment:
        MYSQL_ROOT_PASSWORD: 123456
      volumes:
        - "${LOCAL_PATH}/mysql/slaver-2/data:/var/lib/mysql"
        - "${LOCAL_PATH}/mysql/slaver-2/conf:/etc/mysql/conf.d"
        - "${LOCAL_PATH}/mysql/slaver-2/logs:/var/log/mysql"
      # restart: always
      networks:
        - db
networks:
  db:
```

### 接下来，进行主从配置

```sql
# 进入master容器
docker exec -it mysql-master bash
# 登录MySQL
mysql -uroot -p123456
# 创建一个用户，供同步使用
grant all privileges on *.* to 'slaver'@'%' identified by "123456";
# 刷新权限
flush privileges;
# 查看master同步状态，获取master_log_file和master_log_pos的值
show master status

# 分别进入slaver容器
# 关闭slave同步进程
stop slave
# 连接master，master_host是master的ip，master_user和master_password是在master中创建的用户，master_log_file和master_log_pos的值为从master中获取的。
change master to master_host='mysql-master', master_port=3306, master_user='slaver', master_password='123456', master_log_file='mysql-bin.000004', master_log_pos=624;
# 启动slave同步进程
start slave
# 查看slave同步状态
show slave status
```

