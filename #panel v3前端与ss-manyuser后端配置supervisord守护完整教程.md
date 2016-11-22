#panel v3前端与ss-manyuser后端配置supervisord守护完整教程

###环境 
Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-100-generic x86_64)

LAMP环境安装

```
主要参考:
http://blog.csdn.net/hegaoye308444582/article/details/51980506
```

###常用命令 和 前置安装
更新源
apt-get update

查看当前所有进程
ps aux

搜索安装源
apt-cache search mysql

删除软件

		如果你知道要删除软件的具体名称，可以使用
		sudo apt-get remove --purge 软件名称  
		sudo apt-get autoremove --purge 软件名称 


安装软件

		apt-get install git
		sudo apt-get install apache2
		sudo apt-get install tomcat8 
		apt-get install mysql-server
		sudo apt-get install zip

PHP7.0

		Ubuntu14.04下的默认源是PHP5.0，所以也需要添加外部源
		Personal Package Archive(PPA) 是一个apt仓库，允许第三方开发者发布用于ubuntu的外部资源
		Ondřej Surý 提供了用于PPA的PHP7.0
		sudo apt-get install software-properties-common
		sudo add-apt-repository ppa:ondrej/php
		sudo apt-get update
		sudo apt-get install php7.0
		
git 配置全局github 用户名密码

```
1：cd 回车；进入当前用户目录下；
2：vim .git-credentials （如果没有安装vim 用其它编辑器也可以或 sudo apt-get install vim 安装一个）
3：按照以下格式输入内容：
https://{username}：{password}@github.com
其中username，password，github.com 都换成你自己的内容
4：保存退出后执行下面命令
git config --global credential.helper store
执行完后
/home/用户名/.gitconfig 会新增一项
helper = store
这是再执行git push/pull的时候就不会在要求你输入密码了。
```
mysql 开启远程连接

```
vim /etc/my.cnf
注释这一行：bind-address=127.0.0.1 ==> #bind-address=127.0.0.1
保存退出。
mysql -uroot -p123456
为需要远程登录的用户赋予权限：
复制代码 代码如下:

mysql> GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "123456";
mysql> flush privileges;
```



整合PHP和Apache  

```
sudo apt-get install libapache2-mod-php7.0
sudo service apache2 restart
```

显示PHP的版本信息

```
php -v
Apache默认的网站根目录位于/var/www/html/,进入这个目录，并创建info.php
sudo nano /var/www/html info.php

写入
<?php
phpinfo();
?>
访问该文件查看php是否安装成功
```

###Apache配置

```
AllowOverride参数就是指明Apache服务器是否去找.htacess文件作为配置文件
vim /etc/apache2/apache2.conf
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride All
		Order allow,deny
		Allow from all
        Require all granted
</Directory>

配置二级域名,并指向ss的public路径
vim /etc/apache2/sites-enabled/000-default.conf
<VirtualHost *:80>
       serverName ss.panzijiang.com
        ServerAdmin webmaster@localhost
       DocumentRoot /var/www/ss/public
        ErrorLog ${APACHE_LOG_DIR}/error_ss.log
        CustomLog ${APACHE_LOG_DIR}/access_ss.log combined
</VirtualHost>

开启代理模块
$ sudo ln /etc/apache2/mods-available/proxy.load /etc/apache2/mods-enabled/proxy.load
$ sudo ln /etc/apache2/mods-available/proxy_http.load /etc/apache2/mods-enabled/proxy_http.load

```

###安装步骤

```
一，安装ss-panel
首先我们进入网站目录

cd /home/wwwroot/ss.iforday.com
git clone https://github.com/orvice/ss-panel.git

cp -R ss-panel/. ./
cp .env.example .env

vim .env
=======================
//  修改 ss-panel v3 配置
key = 'randomKey'
env = 'prod'  // 正式环境请保持env为prod确保安全
debug =  'false'  //  正式环境请确保为false
appName = 'ss-panel3'             //站点名称
baseUrl = 'https://www.google.com'            // 站点地址
timeZone = 'PRC'        // RPC 天朝时间  UTC 格林时间
pwdMethod = 'md5'       // 密码加密   可选 md5,sha256
salt = ''               // 密码加密用，从旧版升级请留空
theme    = 'default'   // 主题
// v3.4 后使用 session代替authDriver
// session,cache  可选 file/redis
session = 'file'
cache   = 'file'
tokenDriver = 'db'
// mu key 用于校验ss-go mu的请求
muKey = ''
// 邮件
mailDriver = 'mailgun'   // mailgun   #smtp不在支持,仅供测试

// 注册限制,每天每个ip能注册的次数
ipDayLimit = '10'

// 邮箱验证设置
emailVerifyEnabled = 'false' // 是否开启注册时邮箱验证 (true:开启 false:关闭)
emailVerifyCodeLength = '8'  // 邮箱验证代码长度
emailVerifyTTL = '30'        // 验证代码有效时间 单位分钟

// 用户签到设置
checkinTime = '22'      // 签到间隔时间 单位小时
checkinMin = '93'       // 签到最少流量 单位MB
checkinMax = '97'       // 签到最多流量

//
defaultTraffic = '5'      // 用户初始流量 单位GB
// 注册后获得的邀请码数量
inviteNum = '5'

// 记录流量日志到dynamodb ,beta,请勿开启
log_traffic_dynamodb = 'false'
# database 数据库配置
db_driver = 'mysql'
db_host = 'localhost'
db_port = '3306'
db_database = 'sspanel'
db_username = 'sspanel'
db_password = 'sspanel'
db_charset = 'utf8'
db_collation = 'utf8_general_ci'
db_prefix = ''
=======================

安装composer  php的依赖管理

curl -sS https://getcomposer.org/installer | php
php composer.phar install

把根目录下的db.sql(文件名不一定，大致这样)导入数据库

添加管理员账号:
php xcat createAdmin

按提示填写信息即可

然后给文件权限:

chmod -R 777 storage

然后去参考 上文的Apache配置 配置二级域名,/public目录和代理模块

这个时候就通过二级域名可以打开安装好的sspanel

二，安装shadowsocks manyuser

git clone -b manyuser https://github.com/mengskysama/shadowsocks-rm.git

apt-get install python-pip -y

安装cysql:
pip install cymysql

进入文件夹:
cd shadowsocks-rm
cd shadowsocks

修改config:
vim config.py
=====================

#Config
MYSQL_HOST = 'localhost'
MYSQL_PORT = 3306
MYSQL_USER = 'root'
MYSQL_PASS = ''
MYSQL_DB = ''

MANAGE_PASS = 'passwd'
#if you want manage in other server you should set this value to global ip
MANAGE_BIND_IP = '127.0.0.1'
#make sure this port is idle
MANAGE_PORT = 23333

PANEL_VERSION = 'V3' # V2 or V3. V2 not support API
API_URL = 'http://ss.panzijiang.com/mu'
API_PASS = 'sspanel'
NODE_ID = '1'
CHECKTIME = 15
SYNCTIME = 600

#BIND IP
#if you want bind ipv4 and ipv6 '[::]'
#if you want bind all of ipv4 if '0.0.0.0'
#if you want bind all of if only '4.4.4.4'
SS_BIND_IP = '0.0.0.0'
SS_METHOD = 'rc4-md5'

#LOG CONFIG
LOG_ENABLE = False
LOG_LEVEL = logging.DEBUG
LOG_FILE = '/var/log/shadowsocks.log'

=========================================
通过screen保持他在后台运行
screen -S ss
python servers.py

将配置文件中的log_enadle=true
就可以在/var/log/shadowsocks.log看到运行日志
看到日志 db loop 既shadowsocks也运行起来了
```

###tomcat配置

```
默认目录 /var/lib/tomcat7/webapps/ROOT/index.html

```


启动 shadowsocks servers.py

```
python /var/www/ss/shadowsocks-rm/shadowsocks/servers.py  
```


解压

```
zip -r archive_name.zip directory_to_compress
下面是如果解压一个zip文档：
unzip archive_name.zip

TAR
Tar是在Linux中使用得非常广泛的文档打包格式。它的好处就是它只消耗非常少的CPU以及时间去打包文件，他仅仅只是一个打包工具，并不负责压缩。下面是如何打包一个目录：
tar -cvf archive_name.tar directory_to_compress

如何解包：
tar -xvf archive_name.tar.gz
上面这个解包命令将会将文档解开在当前目录下面。当然，你也可以用这个命令来捏住解包的路径：
tar -xvf archive_name.tar -C /tmp/extract_here/

TAR.GZ
这种格式是我使用得最多的压缩格式。它在压缩时不会占用太多CPU的，而且可以得到一个非常理想的压缩率。使用下面这种格式去压缩一个目录：
tar -zcvf archive_name.tar.gz directory_to_compress
解压缩：
tar -zxvf archive_name.tar.gz
上面这个解包命令将会将文档解开在当前目录下面。当然，你也可以用这个命令来捏住解包的路径：
tar -zxvf archive_name.tar.gz -C /tmp/extract_here/

TAR.BZ2
这种压缩格式是我们提到的所有方式中压缩率最好的。当然，这也就意味着，它比前面的方式要占用更多的CPU与时间。这个就是你如何使用tar.bz2进行压缩。
tar -jcvf archive_name.tar.bz2 directory_to_compress
上面这个解包命令将会将文档解开在当前目录下面。当然，你也可以用这个命令来捏住解包的路径：
tar -jxvf archive_name.tar.bz2 -C /tmp/extract_here/
```




