# panda 熊猫  后端  


## 前言

> 作者: 何全，github地址: https://github.com/hequan2017   QQ交流群: 620176501

> 通过此教程完成从零入门，能够独立编写一个简单的CMDB系统。

> 目前主流的方法开发方式，分为2种：mvc 和 mvvc方式。本教程为 mvc 方式 和 mvvc(前后端分离)的入门教程。


## 教程

![DEMO](images/demo1.png)
![DEMO](images/2-1.png)

[Django之入门 CMDB系统  (一) 基础环境](1.md)

[Django之入门 CMDB系统  (二) 前端模板](2.md)

[Django之入门 CMDB系统  (三) 登录注销](3.md)

[Django之入门 CMDB系统  (四) 增删改查](4.md)

[Django之入门 CMDB系统  (五) 前后端分离之前端](5.md)

[Django之入门 CMDB系统  (六) 前后端分离之后端](6.md)


## 环境介绍

* mvc 模式
* centos 7.6
* python 3.6
* django 2.2
* mysql 5.7
* pycharm 2019.2 (在windows 上 远程centos进行开发)
* vmware workstation 15.5.0
* 项目名字: husky
* cbv 编程方式

## 前端地址
> https://github.com/hequan2017/panda 


## 远端环境配置

* 安装centos 7.6系统
* 安装python3.6
```python 
yum install epel-release -y

yum -y install sqlite  sqlite-devel
yum install  python-devel mysql-devel  python36-devel.x86_64  -y

sudo yum -y install https://centos7.iuscommunity.org/ius-release.rpm

yum install python36  python36-setuptools  -y
easy_install-3.6 pip
python3.6  -m  pip  install   --upgrade  pip


mv   /usr/bin/python  /tmp/
ln -s /usr/bin/python3.6    /usr/bin/python

sed  -i    's/\#\!\/usr\/bin\/python/\#\!\/usr\/bin\/python2/'   /usr/bin/yum
sed  -i    's/\#\! \/usr\/bin\/python/\#\! \/usr\/bin\/python2/'   /usr/libexec/urlgrabber-ext-down


mkdir -p  /root/.pip/

cat >  /root/.pip/pip.conf   <<EOF
[global]
trusted-host=mirrors.aliyun.com
index-url=http://mirrors.aliyun.com/pypi/simple/
EOF

python -V
```

* 安装django   

> pip3 install django==2.2.6

## 本地开发配置

* 配置pycharm

> file-->settings-->project interpreter--> add --> ssh interpreter 设置远端 python环境

> 设置/usr/bin/python3.6   目录选择 <Project root>→/opt

> file--> new project --> django 

![DEMO](images/1-1.png)

## 部署数据库

* 为了快速，采用docker方式部署。
```shell script
mkdir -p  /data/docker
mkdir -p /data/mysql5722

sudo yum install -y yum-utils device-mapper-persistent-data lvm2

sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

sudo yum makecache fast
sudo yum -y install docker-ce


docker version
systemctl enable docker.service    
systemctl start docker.service



sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{"registry-mirrors": ["https://890km4uy.mirror.aliyuncs.com"],"graph": "/data/docker"}
EOF
sudo systemctl daemon-reload


sudo systemctl restart docker
```
```shell script
mkdir -p /data/mysql5722
mkdir -p /data/mysql5722-cnf

docker run -itd \
--name mysql \
-p 3306:3306 \
--mount type=bind,src=/data/mysql5722,dst=/var/lib/mysql \
--mount type=bind,src=/data/mysql5722-cnf,dst=/etc/mysql \
-e MYSQL_ROOT_PASSWORD=123456  \
mysql:5.7.22 --character-set-server=utf8


yum remove mariadb  -y 
yum install  mariadb    -y
mysql -uroot  -p123456 -h 192.168.100.99
create database  husky;

```

## 配置django 数据库
> husky --> settings

```shell script
ALLOWED_HOSTS = ['*']   ##允许所有地址访问


DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'HOST': '192.168.100.99',
        'PORT': '3306',
        'NAME': 'husky',
        'USER': 'root',
        'PASSWORD': '123456',
    }
}

# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.sqlite3',
#         'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
#     }
# }
```

* 修改python文件

```shell script

vim /usr/local/lib64/python3.6/site-packages/django/db/backends/mysql/base.py

35 #if version < (1, 3, 13):   注释掉 这两行
36 #    raise ImproperlyConfigured('mysqlclient 1.3.13 or newer is required; you have %s.' % Database.__version__)   


vim /usr/local/lib64/python3.6/site-packages/django/db/backends/mysql/operations.py

145         if query is not None:
146             query = query.encode(errors='replace')   ##修改此行

```

* 执行数据库初始化
> pycharm : 菜单栏 tools --> 选择  run  manage.py task

> makemigrations    生成数据文件
 
> migrate           根据文件，执行生成表结构
 
> createsuperuser

---

> 设置pycharm  项目启动 地址 为 192.168.100.99

> pycharm 启动django项目 (非命令行启动)

```shell script
#pycharm显示信息
ssh://root@192.168.100.99:22/usr/bin/python3.6 -u /opt/manage.py runserver 192.168.100.99:8000
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
October 31, 2019 - 09:33:33
Django version 2.2.6, using settings 'husky.settings'
Starting development server at http://192.168.100.99:8000/
Quit the server with CONTROL-C.
```


 
 
