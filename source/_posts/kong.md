---
title: kong
date: 2018-12-02 19:02:17
tags:
---

当前系统，环境centos7
起kong服务需要做：
- 安装
    - kong
    - pgsql
    - dashboard(可选，用来通过api来管理空)
- 配置
    - kong.conf（配置数据库等）、配置路由。
    - pgsql配置（新建数据库）
    - 配置kong路由
- 注意事项
    - 尽量安装高版本，防止和pgsql不兼容。


# 1.安装kong

- 需要安装kong
- 安装pgsql

官方文档提供的快速安装地址`https://docs.konghq.com/install/centos/?_ga=2.182378343.1594956911.1543481061-735966922.1539239867`

step1:
`baseurl=https://kong.bintray.com/kong-community-edition-rpm/centos/7` 各个版本的地址，下载要安装的版本。
wget `https://kong.bintray.com/kong-community-edition-rpm/centos/7/kong-community-edition-0.14.1.el7.noarch.rpm`
srep2:
执行下边命令:
```
sudo yum install epel-release
sudo yum install kong-community-edition-0.14.1.*.noarch.rpm --nogpgcheck
```
安装完成，这个时候是起不来额，因为没有安装pgsql。


# 2.安装postgresql10

尽量安装高版本，防止和pgsql不兼容,为了避免采坑，选择`kong-community-edition-0.14.1.el7.noarch.rpm`版本，当然也在测试机测试没有问题。
官方地址`https://www.postgresql.org/download/linux/redhat/` 可以选择对需安装的对应的版本，会自动生成安装命令，我选择的是10.6

postgresql10安装与启动
```
yum install https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm    
yum install postgresql10    
yum install postgresql10-server
/usr/pgsql-10/bin/postgresql-10-setup initdb  
systemctl enable postgresql-10  
systemctl start postgresql-10
```
安装完成后，还需要做：
1. 配置postgreps用户。
2. 新增和数据库。
3. 配置远程连接。

添加用户以及新增数据：
```
sudo -s  //取得root权限
su - postgres 
psql -U postgres //连接本地数据

postgres=# CREATE USER kong WITH LOGIN PASSWORD 'kong';  //创建用户
postgres-# CREATE ROLE //返回CREATE ROLE，表示创建用户成功

postgres=# CREATE DATABASE kong; //创建数据库
CREATE DATABASE //返回CREATE DATABASE，表示创建数据库成功.

```

然后初始化数据库:`sudo /usr/local/bin/kong migrations up`

修改pgsql,配置远程连接：
执行`vim /var/lib/pgsql/10/data/pg_hba.conf`
```
//修改 127.0.0.1/32 为trust
host    all             all             127.0.0.1/32            trust
//添加下边这条记录，用于ssl连接
host    all             all             0.0.0.0/0               md5
```
![image](/photo/img/kong/pg_hba.conf.png)

执行`vim /var/lib/pgsql/10/data/postgresql.conf`
添加一下信息:
```
listen_addresses = '*' //允许所有的ip访问postgres
```

# 3.配置kong.conf

执行 `vi /etc/kong/kong.conf`

```
pg_host = 127.0.0.1             # The PostgreSQL host to connect to.
pg_port = 5432                  # The port to connect to.
pg_user = kong                  # The username to authenticate if required.
pg_password = kong                  # The password to authenticate if required.
pg_database = kong              # The database name to connect to.
```
执行`sudo /usr/local/bin/kong start -c /etc/kong/kong.conf`,提示`Kong started`则代表启动成功

`curl -i http://localhost:8001/`,测试kong是否启动。也可以访问`http://host:8000/`,来确定kong是否起来。因为此时我们尚未配置任何服务、路由，访问会返回404，如下：
![image](/photo/img/kong/kong-404.png)


# 4.配置dashboard
前置操作：需要安装nodejs
`sudo yum install nodejs`

kong提供一套api，用于动态管服务、路由、插件等。当然有开源的dashboard，这样我们操作起来更方便。
开源项目地址`https://github.com/PGBI/kong-dashboard`

安装命令`npm install -g kong-dashboard`

启动命令：`nohup kong-dashboard start --kong-url http://localhost:8001 /dev/null 2>&1 &`

为了安全起见，可以吧启动kong的账户密码放在work的环境变量里。
修改work的环境变量 vi ~.bash_profile，添加password参数。
kong-dashboard start \
  --kong-url http://kong:8001 \
  --basic-auth work=$password

# 5.日志目录以及启动相关
查看日志文件`tail -f /usr/local/kong/logs/error.log`

kong配置文件默认地址为 /ect/kong/kong.conf
想要删除postgreSql，需要先断开所有连接`sudo ps aux | grep scidb | xargs sudo kill -9`

启动命令`sudo /usr/local/bin/kong start -c /etc/kong/kong.conf`

# 6.遇到问题

沙盒项目部署的时候，我把项目上传到了 /home/work/kong/目录下，并将其软连到了/usr/local/share/lua/目录下。、

再启动kong时，报错:

```
2018/12/06 15:26:01 [error] 7684#0: init_worker_by_lua error: /usr/local/share/lua/5.1/kong/init.lua:244: module 'resty.worker.events' not found:Failed loading module resty.worker.events in LuaRocks rock lua-resty-worker-events 0.3.3-1
	no field package.preload['resty.worker.events']
	no file './resty/worker/events.lua'
	no file './resty/worker/events/init.lua'
	no file '/usr/local/openresty/site/lualib/resty/worker/events.ljbc'
	no file '/usr/local/openresty/site/lualib/resty/worker/events/init.ljbc'
	no file '/usr/local/openresty/lualib/resty/worker/events.ljbc'
	no file '/usr/local/openresty/lualib/resty/worker/events/init.ljbc'
	no file '/usr/local/openresty/site/lualib/resty/worker/events.lua'
	no file '/usr/local/openresty/site/lualib/resty/worker/events/init.lua'
	no file '/usr/local/openresty/lualib/resty/worker/events.lua'
	no file '/usr/local/openresty/lualib/resty/worker/events/init.lua'
	no file '/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/resty/worker/events.lua'
	no file '/usr/local/share/lua/5.1/resty/worker/events.lua'
	no file '/usr/local/share/lua/5.1/resty/worker/events/init.lua'
	no file '/usr/local/openresty/luajit/share/lua/5.1/resty/worker/events.lua'
	no file '/usr/local/openresty/luajit/share/lua/5.1/resty/worker/events/init.lua'
	no file '/root/.luarocks/share/lua/5.1/resty/worker/events.lua'
	no file '/root/.luarocks/share/lua/5.1/resty/worker/events/init.lua'
	no file '/usr/local/openresty/site/lualib/resty/worker/events.so'
	no file '/usr/local/openresty/lualib/resty/worker/events.so'
	no file './resty/worker/events.so'
	no file '/usr/local/lib/lua/5.1/resty/worker/events.so'
	no file '/usr/local/openresty/luajit/lib/lua/5.1/resty/worker/events.so'
	no file '/usr/local/lib/lua/5.1/loadall.so'
	no file '/root/.luarocks/lib/lua/5.1/resty/worker/events.so'
	no file '/usr/local/openresty/site/lualib/resty.so'
	no file '/usr/local/openresty/lualib/resty.so'
	no file './resty.so'
	no file '/usr/local/lib/lua/5.1/resty.so'
	no file '/usr/local/openresty/luajit/lib/lua/5.1/resty.so'
	no file '/usr/local/lib/lua/5.1/loadall.so'
	no file '/root/.luarocks/lib/lua/5.1/resty.so'
```

最终我把项目上传到了/usr/local/share/releas/目录下，并将其软连到了/usr/local/share/lua/下，启动并没有出现改问题.


线索：
`https://github.com/Kong/kong/issues/2838`
