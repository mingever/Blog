---
title: Docker 常用软件
date: 2025-04-09 11:59:35
cover:
top_img:
tags:
- 搭建
categories:
- 搭建
keywords:
description: docker搭建环境经常使用的软件简单记录一下
---

wsl2目前已经很完善了，wsl+docker对个人搭建开发环境来说实在是太方便了，用了就离不开

### Portainer

``` bash
docker run --name portainer -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer-ce
#本地挂在路径必须为/var/run/docker.sock，才可以连接上本地环境，不要改成别的
```

### Mysql

```bash
docker run -d -p 3306:3306 -v /root/mysql/conf:/etc/mysql/conf.d -v /root/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=169736 --name mysql mysql:8.0
```
设置开机不启动，并设置大小写不敏感

```bash
docker run -d -p 3306:3306 -v /root/mysql/conf:/etc/mysql/conf.d -v /root/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=169736 --name mysql mysql:8.0 --lower_case_table_names=1
```

```bash
# 进入容器内mysql配置
docker exec -it mysql mysql -u root -p
```

### Oracle

```bash
docker pull marcocimatti/oracle12.2.0.1-se2
```

```bash
docker run -d -it --name oracle \
-p 1521:1521 -p 5500:5500 \
-e ORACLE_SID=orcl \
-e ORACLE_PDB=orclPDB1 \
-e ORACLE_PWD=169736 \
-e ORACLE_EDITION=standard \
-e ORACLE_CHARACTERSET=AL32UTF8 \
-v /root/oracle/data:/opt/myData/oracle/oracleData \
marcocimatti/oracle12.2.0.1-se2

#ORACLE_SID=orcl 是服务名，安装后连接Oracle 时用的服务名，CDB数据库的
#ORACLE_PWD=123456 是安装后 dba 用户的密码，即：123456
#ORACLE_PDB=orclPDB1 是安装后的PDB数据库，想更明白一点，可以往下看6.3.1创建用户的两种方式，一个是在CDB库下，一个是zaiPDB库下
```

### Nginx

**1）拉取**

```  bash
docker pull nginx
```

**2）创建nginx配置文件**

先创建一个临时容器，然后从容器中复制 default.conf出来，可以自行修改default.conf，自定义配置项。
然后再创建正式使用的 nginx 容器并挂载。
将容器文件拷贝到主机中命令：`docker cp 容器id:/容器文件路径/文件名 /主机文件路径`

``` bash
dockre cp nginx:/etc/nginx/conf.d/default.conf /root/nginx/conf
```



docekr中nginx的默认路径位置

- conf：/etc/nginx/nginx.conf

- html：/usr/share/nginx/html

	网站文件存放的地方, 默认只有Nginx欢迎页面, 可以通过改变Nginx配置文件的方式来修改这个位置

- log：/var/log/nginx

	access.log：每一个访问请求都会默认记录在这个文件中

	error.log：  任何Nginx的错误信息都会记录到这个文件中

> nginx容器/etc/nginx/conf.d文件夹内，自带default.conf配置文件。和redis不一样，redis容器内是不带配置文件的
>
> 直接挂载文件夹的话，如果宿主机内为空文件夹，那么容器内挂载的文件夹里的内容将会被清空
> 如果宿主机内为非空文件夹的话，那么容器内挂载的文件夹里的内容将会被清空，再将宿主机内的文件复制到容器内的文件夹里
>
> 直接挂载文件的话（docker可以直接挂载文件），用vim修改宿主机文件后，只有重启容器后，容器内文件内容才会变化。要想不重启，需要对容器内的文件赋予777权限，`chmod 777 redis.conf`。所以挂载文件夹会方便些。
>
> 挂载后，改变宿主机文件，容器内文件会变化；改变容器内文件，宿主机内文件同样会变化
>
> (docker在进行文件挂载时，并不是仅仅挂载文件名到对应位置，而是将文件对应的inode 进行映射。用vim进行文件的编辑并保存时，系统采用的是备份、替换的策略，文件用vim等工具编辑的过程实质是，备份原来的文件，当新文件编辑完成后，再将新文件替换文原件，这会导致文件的inode变化，所以docker内外的文件并不会同步。而用cat，echo等重定向操作修改文件时，文件的inode保持不变，所以不会发生类似现象。)

**3）创建nginx容器并运行**

```bash
docker run -d \
--name nginx \
-p 80:80 \
-v /root/nginx/conf:/etc/nginx/conf.d \
-v /root/nginx/html:/usr/share/nginx/html \
-v /root/nginx/log:/var/log/nginx \
nginx

#/usr/share/nginx/html为nginx，网站文件存放的地方, 默认只有Nginx欢迎页面。挂载后即可替换为自己的网站页面
```

### Rabbit

1. **拉取**

	在[docker hub](https://hub.docker.com/)里搜索rabbitmq，选择带有mangement的版本

	> 因为国内镜像下架，rabbitmq大部分版本拉不下来了，可以试试rabbitmq:3.8-management

	```bash
	docker pull rabbitmq:3.11.9-management
	```

2. **安装**

	```bash
	docker run \
	-d --name rabbitmq \
	-p 5672:5672 \
	-p 15672:15672 \
	-v /root/rabbitmq:/var/lib/rabbitmq \
	--hostname myRabbit \
	-e RABBITMQ_DEFAULT_VHOST=my_vhost \
	-e RABBITMQ_DEFAULT_USER=admin \
	-e RABBITMQ_DEFAULT_PASS=169736 \
	rabbitmq:3.11.9-management
	
	#-p 指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）
	#--hostname  主机名（RabbitMQ的一个重要注意事项是它根据所谓的 “节点名称” 存储数据，默认为主机名）
	#-e 指定环境变量，默认虚拟机名，默认用户名和默认用户名密码
	```

### Redis

1. 拉取

	```bash
	docker pull redis
	```

2. 安装

	  [配置文件，官网下载](https://redis.io/docs/latest/operate/oss_and_stack/management/config/)

	```bash
	docker run --restart=always \
	-d -p 6379:6379 \
	--name redis \
	-v /root/redis/conf/redis.conf:/etc/redis/redis.conf \
	-v /root/redis/data:/data \
	redis:6.0.16 \
	redis-server /etc/redis/redis.conf \
	--requirepass 169736
	
	#说明：
	#–restart=always                     总是开机启动
	#redis-server /etc/redis/redis.conf  以配置文件启动redis，加载容器内的conf文件
	#–appendonly yes                     开启aof持久化，默认是rdb
	#–requirepass 169736                 设置密码(向外开放，一定要设置密码，否则会有挖矿木马)
	#直接创建的redis容器里是没有配置文件的，该配置文件是需要在创建容器时映射进来的，需要从官网下载。和nginx容器内自带配置文件不一样
	#可以使用curl -O https://raw.githubusercontent.com/redis/redis/6.0/redis.conf 来将文件下载到当前目录
	#redis.conf中，daemoinze要设置为no，禁止后台启动，否咋会与docker的 -d发送冲突
	#-v /root/redis-demo/conf:/etc/redis 也可以把配置文件复制到文件夹里，然后直接挂载文件夹。推荐挂载文件夹，挂载文件的话，因为inode映射，需要为宿主文件赋予777权限才不需要重启，chmod 777 redis.conf。
	```

### Remark

#### restart

restart参数用于指定自动重启docker容器策略，包含3个选项：`no`，`on-failure[:times]`，`always`，`unless-stopped`

- no 默认值，表示容器退出时，docker不自动重启容器

	```bash
	docker run --restart=no [容器名]
	```

- on-failure 若容器的退出状态非0，则docker自动重启容器，还可以指定重启次数，若超过指定次数未能启动容器则放弃

	```bash
	docker run --restart=on-failure:3 [容器名]
	```

- always 容器退出时总是重启

	```bash
	docker run --restart=always [容器名]
	```

- unless-stopped 容器退出时总是重启，但不考虑Docker守护进程启动时就已经停止的容器

	```bash
	docker update --restart=always [容器名]
	```

如果容器启动时没有设置–restart参数，则通过下面命令进行更新：

```bash
docker update --restart=always [容器名]
```

#### 挂载文件和文件夹

[docker挂载文件和文件夹相关问题 ](https://segmentfault.com/a/1190000015684472#:~:text=)

- 文件夹：
	host上文件夹一定会覆盖container中文件夹

- 文件：

	文件挂载与文件夹挂载最大的不同点在于：

	- docker 禁止用主机上不存在的文件挂载到container中已经存在的文件
	- 文件挂载不会对同一文件夹下的其他文件产生任何影响

	除此之外， 其覆盖行为与文件夹挂载一致，即
	用host上的文件的内容覆盖container中的文件的内容
