# 临兵漏洞扫描系统

> 本系统是对目标进行漏洞扫描的一个系统,前端采用vue技术,后端采用flask.核心原理是扫描主机的开放端口情况,然后根据端口情况逐个去进行poc检测,poc有110多个,包含绝大部分的中间件漏洞,本系统的poc皆来源于网络或在此基础上进行修改

## 打包vue源代码(进入到vue_src目录下)

> npm run build(有打包好的,即vue文件夹,可直接使用,自行打包需要安装node和vue,参考<https://www.runoob.com/nodejs/nodejs-install-setup.html>, <https://www.runoob.com/vue2/vue-install.html>)

## ubuntu部署(强烈建议在ubuntu上搭建,centos源中无python3.8,在后面自制uwsgi插件时,还得重新编译gcc,特别麻烦,费时间)

### 设置国内源

> sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list && apt-get clean && apt update

### 安装依赖

> apt install -y mariadb-server python3.8 python3.8-dev python3-pip uwsgi uwsgi-src nmap masscan nginx libpq-dev uuid-dev libcap-dev libpcre3-dev python3-dev inetutils-ping

> mkdir /root/flask && mkdir /var/log/uwsgi

### 设置python3.8为python3

> update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 1 && update-alternatives --config python3

### 安装python3依赖库

> pip3 install -r /root/flask/requirements.txt

> 如果你使用的是低于python3.8版本的python3,请把run.py文件中第16行注释去掉,并注释掉第17行

### nginx

#### 启动nginx

> nginx

#### 添加nginx用户

> useradd -s /sbin/nologin -M nginx

#### 配置

> uwsgi配置文件已配置好,可以直接使用,可以根据自己的需求修改文件路径及端口.conf.ini文件中配置数据库信息和邮件配置

> 在/etc/nginx/conf.d目录下放入flask.conf和vue.conf文件

> 在/etc/nginx目录下放入nginx.conf文件

> conf配置文件中有注释

> 把vue目录移到/usr/share/nginx/html中

> 在flask/conf.ini中配置数据库和发送邮件设置

### mariadb

#### 启动mariadb

> service mysql start

#### 设置mariadb密码(password为你要设置的密码)

> mysql -e "SET PASSWORD FOR root@localhost = PASSWORD('password');FLUSH PRIVILEGES;"

> mysql -e "update mysql.user set plugin='mysql_native_password' where User='password';FLUSH PRIVILEGES;"

> 配置数据库密码后需要在flask/conf.ini文件中配置连接maridab数据库的用户名,密码等信息

### 邮件

> 我使用的是QQ邮箱发送的邮件,需要授权码,需要自行到flask/conf.ini文件中去设置,参考<https://blog.csdn.net/Momorrine/article/details/79881251>

#### 执行uwsgi脚本(自制uwsgi-plugin-python38, ubuntu系统目前最高支持uwsgi-plugin-python36)

> chmod +x ubuntu_uwsgi.sh

> ./ubuntu_uwsgi.sh

#### 配置uwsgi

> 把uwsgi.ini文件放到flask文件夹的根目录下(我的flask文件夹路径是/root/flask,如果各位不是这个路径,需要到uwsgi.ini文件和flask.conf中修改文件的路径)

#### 启动uwsgi

> 进入到/root/flask/目录下,uwsgi --ini uwsgi.ini(必须进到相关目录中执行)

## centos部署(强烈建议在ubuntu上搭建,centos源中无python3.8,在后面自制uwsgi插件时,还得重新编译gcc,特别麻烦,费时间)

### 设置源

> mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup && curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo && curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo && sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo yum clean all && yum makecache && yum update -y

### 安装依赖

> yum install -y epel-release mariadb-server gcc gcc-c++ wget bzip2 zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel make libffi-devel nmap  masscan  nginx initscripts postgresql-devel python3-devel uwsgi uwsgi-plugin-common 

> mkdir /root/flask && mkdir /var/log/uwsgi 

### 重新编译gcc

> wget https://ftp.gnu.org/gnu/gcc/gcc-9.2.0/gcc-9.2.0.tar.xz && tar xvf gcc-9.2.0.tar.xz

> cd gcc-9.2.0 && ./contrib/download_prerequisites && mkdir build && cd build && ../configure --prefix=/usr/local --disable-multilib --enable-languages=c,c++ && make && make install 

> ln -sf /usr/local/bin/gcc cc && yum remove -y gcc

### 安装python3.8

> wget https://www.python.org/ftp/python/3.8.1/Python-3.8.1.tgz 

> tar -zxvf Python-3.8.1.tgz

> cd Python-3.8.1 && ./configure prefix=/usr/local/python3.8 --enable-shared --enable-optimizations LDFLAGS="-Wl,--rpath=/usr/local/python3.8/lib" make && make install

> rm -rf /usr/bin/python3 && rm -rf /usr/bin/pip3

> ln -s /usr/local/python3.8/bin/python3.8 /usr/bin/python3 && ln -s /usr/local/python3.8/bin/pip3.8 /usr/bin/pip3

### 安装python3依赖库

> pip3 install -r /root/flask/requirements.txt

> 如果你使用的是低于python3.8版本的python3,请把run.py文件中第16行注释去掉,并注释掉第17行

### nginx

#### 启动nginx

> systemctl start nginx

#### 添加nginx用户

> useradd -s /sbin/nologin -M nginx

#### 配置

> 配置文件已配置好,可以直接使用,可以根据自己的需求修改文件路径及端口.

> 在/etc/nginx/conf.d目录下放入flask.conf和vue.conf文件

> 在/etc/nginx目录下放入nginx.conf文件

> conf配置文件中有注释

> 把vue目录移到/usr/share/nginx/html中

> 在flask/conf.ini中配置数据库和发送邮件设置

### mariadb

#### 启动mariadb

> systemctl start mariadb

#### 进行数据库配置(如设置密码等)

> mysql_secure_installation(具体步骤略去,可参考<https://www.cnblogs.com/yhongji/p/9783065.html>)
> 配置数据库密码后需要在flask/conf.ini文件中配置连接maridab数据库的用户名,密码等信息

### 邮件

> 我使用的是QQ邮箱发送的邮件,需要授权码,需要自行到flask/conf.ini文件中去设置,参考<https://blog.csdn.net/Momorrine/article/details/79881251>

#### 执行uwsgi脚本(自制uwsgi-plugin-python38, centos系统目前最高支持uwsgi-plugin-python36)

> chmod +x centos_uwsgi.sh

> ./centos_uwsgi.sh

#### 配置uwsgi

> 把uwsgi.ini文件放到flask文件夹的根目录下(我的flask文件夹路径是/root/flask,如果各位不是这个路径,需要到uwsgi.ini文件和flask.conf中修改文件的路径)

#### 启动uwsgi

> 进入到/root/flask/目录下,uwsgi --ini uwsgi.ini(必须进到相关目录中执行)

## docker部署

### 配置

> 首先下载项目到本地(https://github.com/taomujian/linbing.git),然后配置flask/conf.ini中发送邮件所用的账号和授权码,然后修改flask/conf.ini的mysql数据库账号密码,这个账号密码要和dockerfile中的设置的账号密码保持一致

### 编译镜像(进入项目根目录)

> docker build -f ubuntu.dockerfile -t linbing .

### 启动容器(进入项目根目录)

> docker run -it -d -p 11000:11000 linbing 

## 访问

> 访问<http://yourip:11000/login>即可

## CHANGELOG

### [v1.0] 2020.2.28
- 初步完成扫描器功能

### [v1.1] 2020.7.28
- 新增F5 BIG IP插件

### [v1.2] 2020.8.12
- 增加docker部署

### [v1.3] 2020.9.13
- 增加phpstudy_back_rce插件数量
- 添加目标时可添加多行目标

### [v1.4] 2020.10.18
- 增加查看端口详情(端口、协议、产品、版本)
- 增加子域名详情(子域名,子域名ip),子域名是用的oneforall工具

### [v1.5] 2020.10.30
- 修改一些插件的错误
- 扫描设置中可设置POC检测时协程的并发数量
- 增加asyncio多协程功能,提高POC扫描速度

## 致谢

> 感谢vulhub项目提供的靶机环境:<https://github.com/vulhub/vulhub>,还有<https://hub.docker.com/r/2d8ru/struts2>

> POC也参考了很多项目:<https://github.com/Xyntax/POC-T>、<https://github.com/ysrc/xunfeng>、<https://github.com/se55i0n/DBScanner>、<https://github.com/vulscanteam/vulscan>

> 感谢师傅pan带我入门安全,也感谢呆橘同学在vue上对我的指导

## 免责声明

工具仅用于安全研究以及内部自查，禁止使用工具发起非法攻击，造成的后果使用者负责
