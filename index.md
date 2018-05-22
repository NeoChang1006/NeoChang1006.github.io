your must run：
```
shell> yum install -y make gcc gcc-c++ perl zlib-devel libaio libpng libpng-devel libjpeg-devel pcre-devel
shell> yum install -y libXpm-devel openssl openssl-devel libxml2-devel bzip2-devel.x86_64 libjpeg-turbo-devel
```
unnecessary run：
```
yum install -y freetype freetype-devel libtool cmake ncurses-devel bison re2c curl-devel wget
rpm -ivh "http://mirrors.sohu.com/fedora-epel/epel-release-latest-6.noarch.rpm"
yum install -y libmcrypt-devel re2c
```

first part：Install Nginx
Document：https://www.nginx.com/resources/wiki/start/topics/tutorials/install/
一、Official CentOS packages
To add NGINX yum repository, create a file named ```/etc/yum.repos.d/nginx.repo``` and paste one of the configurations below:
CentOS:
```
shell> vim /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

二、Source Releases
wget Stable NGINX
```
shell> cd /usr/local/src
shell> wget http://nginx.org/download/nginx-1.10.1.tar.gz
shell> tar xf nginx-1.10.1.tar.gz
```

三、Building NGINX From Source
After extracting the source, run these commands from a terminal:
```
shell> cd nginx-1.10.1/
shell> ./configure
return 
  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
shell> make
shell> make install
```

四、Starting, Stopping, and Restarting NGINX
/usr/local/nginx/sbin
Basic Example of Starting NGIN
```
shell> ln -s  /usr/local/nginx/sbin/nginx  /usr/bin/nginx
shell> /usr/bin/nginx
```


second part：Install PHP
Document：http://php.net/manual/zh/install.unix.nginx.php
一、wget Stable PHP
```
shell> cd /usr/local/src
shell> wget http://cn2.php.net/distributions/php-7.2.4.tar.gz
shell> tar xf php-7.2.4.tar.gz
```

二、configure and build PHP. 
This is where you customize PHP with various options, like which extensions will be enabled. Run ./configure --help for a list of available options. In our example we'll do a simple configure with PHP-FPM and MySQLi support.
```
shell> cd php-7.2.4
shell> ./configure --enable-fpm --with-mysqli
shell> make
shell> make test
shell> make install
return 
Installing shared extensions: /usr/local/lib/php/extensions/no-debug-non-zts-20170718/
Installing PHP CLI binary: /usr/local/bin/
Installing PHP CLI man page: /usr/local/php/man/man1/
Installing PHP FPM binary: /usr/local/sbin/
Installing PHP FPM defconfig: /usr/local/etc/
Installing PHP FPM man page: /usr/local/php/man/man8/
Installing PHP FPM status page: /usr/local/php/php/fpm/
Installing phpdbg binary: /usr/local/bin/
Installing phpdbg man page: /usr/local/php/man/man1/
Installing PHP CGI binary: /usr/local/bin/
Installing PHP CGI man page: /usr/local/php/man/man1/
Installing build environment: /usr/local/lib/php/build/
Installing header files: /usr/local/include/php/
Installing helper programs: /usr/local/bin/
  program: phpize
  program: php-config
Installing man pages: /usr/local/php/man/man1/
  page: phpize.1
  page: php-config.1
Installing PEAR environment: /usr/local/lib/php/
[PEAR] Archive_Tar - installed: 1.4.3
[PEAR] Console_Getopt - installed: 1.4.1
[PEAR] Structures_Graph- installed: 1.1.1
[PEAR] XML_Util - installed: 1.4.2
[PEAR] PEAR - installed: 1.10.5
Wrote PEAR system config file at: /usr/local/etc/pear.conf
You may want to add: /usr/local/lib/php to your php.ini include_path
/usr/local/src/php-7.2.4/build/shtool install -c ext/phar/phar.phar /usr/local/bin
ln -s -f phar.phar /usr/local/bin/phar
Installing PDO headers: /usr/local/include/php/ext/pdo/
```

三、Obtain and move configuration files to their correct locations
```
shell> cp /usr/local/src/php-7.2.4/php.ini-development /usr/local/lib/php.ini
shell> cp /usr/local/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf
shell> cp /usr/local/src/php-7.2.4/sapi/fpm/php-fpm /usr/local/bin
```

四、modify php.ini
It is important that we prevent Nginx from passing requests to the PHP-FPM backend if the file does not exists, allowing us to prevent arbitrarily script injection.
We can fix this by setting the cgi.fix_pathinfo directive to 0 within our php.ini file.
Load up php.ini:
```
shell> vim /usr/local/lib/php.ini
cgi.fix_pathinfo=0
```
Locate cgi.fix_pathinfo= and modify it as follows

五、run php-fpm
```
shell> groupadd www
shell> useradd www -g www
```
php-fpm.conf must be modified to specify that php-fpm must run as the user ```www``` and the group ```www``` before we can start the service:
vim ```/usr/local/etc/php-fpm.conf```，Find and modify the following:
```
shell> cp /usr/local/etc/php-fpm.d/www.conf.default /usr/local/etc/php-fpm.d/www.conf
shell> vim /usr/local/etc/php-fpm.d/www.conf
[www]
user = www
group = www
```
The php-fpm service can now be started:
```
shell> /usr/local/bin/php-fpm
```

thire part：PHP & Nginx
一、Nginx must now be configured to support the processing of PHP applications:
vim ```/usr/local/nginx/conf/nginx.conf```
Modify the default location block to be aware it must attempt to serve .php files:
```
shell> vim /usr/local/nginx/conf/nginx.conf
location / {
    root html;
    index index.php index.html index.htm;
}
```
The next step is to ensure that .php files are passed to the PHP-FPM backend. Below the commented default PHP location block, enter the following:
```
shell> vim /usr/local/nginx/conf/nginx.conf
location ~ \.php$ {
     root /usr/local/nginx/html;
     fastcgi_index index.php;
     fastcgi_pass 127.0.0.1:9000;
     include fastcgi_params;
     fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```
Restart Nginx.
```
shell> /usr/local/nginx/sbin/nginx -s reload
```
二、Create a test file
```
rm /usr/local/nginx/html/index.html
echo "<?php phpinfo(); ?>" >> /usr/local/nginx/html/index.php
```

fourth part：Install MySQL
eg.https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.22-linux-glibc2.12-x86_64.tar.gz
```
shell> groupadd mysql
shell> useradd -r -g mysql -s /bin/false mysql
shell> cd /usr/local/src
shell> wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.22-linux-glibc2.12-x86_64.tar.gz
shell> tar xf mysql-5.7.22-linux-glibc2.12-x86_64.tar.gz
shell> mv /usr/local/src/mysql-5.7.22-linux-glibc2.12-x86_64 /usr/local/
shell> cd /usr/local
shell> ln -s mysql-5.7.22-linux-glibc2.12-x86_64.tar.gz mysql
shell> cd mysql
shell> mkdir mysql-files
shell> chown -R mysql:mysql mysql-files
shell> chmod 750 mysql-files
shell> /usr/local/mysql/bin/mysqld --initialize --user=mysql //这一步会返回MySQL密码，一定要记住
return 
A temporary password is generated for root@localhost: %:#.wdGNV2C9
shell> /usr/local/mysql/bin/mysql_ssl_rsa_setup
shell> /usr/local/mysql/bin/mysqld_safe --user=mysql &
return
2018-05-04T12:36:49.064297Z mysqld_safe Logging to '/var/log/mysqld.log'.
2018-05-04T12:36:49.082759Z mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2018-05-04T12:36:49.499394Z mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
# Add Environment variables--建议官方文档在安装页面加上此行命令
shell> vim /etc/profile
# Path manipulation
if [ "$EUID" = "0" ]; then
    pathmunge /usr/local/src/mysql/bin
    pathmunge /etc/init.d
else
    pathmunge /usr/local/src/mysql/bin
    pathmunge /etc/init.d
fi
shell> source /etc/profile
shell> ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock
# Next command is optional
shell> cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql.server
shell> mysql.server start
```

if start mysql.server return error message,eg.
```
shell> mysql.server start
/usr/local/src/mysql/support-files/mysql.server: line 239: my_print_defaults: command not found
/usr/local/src/mysql/support-files/mysql.server: line 259: cd: /usr/local/mysql: 没有那个文件或目录
Starting MySQLCouldn't find MySQL server (/usr/local/mysql/[失败]sqld_safe)
```
your have change ```/usr/local/support-files/mysql.server``` files.
```
vim support-files/mysql.server
/usr/local/mysql replace /usr/local/src/mysql
mysql.server start
```
if start mysql.server return error message,eg.
```
shell> mysql.server start
Starting MySQL...The server quit without updating PID file [失败]lib/mysql/sanqianServer.pid).
```
sorry i don't know

if you want to ```mysql ``` 
```
shell> vim /etc/my.cnf
[client]
user=root
password=qkyrYpd2um&)
```
[root@sanqianServer mysql]# 2018-04-21T10:10:50.118966Z mysqld_safe Logging to '/var/log/mysqld.log'.//mysqld日志
2018-04-21T10:10:50.152620Z mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2018-04-21T10:10:51.014194Z mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended



fifth part：Install Redis
1、download redis
```
shell > cd /usr/local/
shell > wget http://download.redis.io/releases/redis-2.8.17.tar.gz
```

2、Install
```
shell > tar xf redis-2.8.17.tar.gz
shell > ln -s redis-2.8.17.tar.gz redis
shell > cd redis
shell > make
return 
Leaving directory `/usr/local/redis-4.0.9/src'
```

3、start redis
```
shell > /usr/local/redis/src/redis-server
```

sixth part：PHP & Redis
一、Install  PHP redis drive
1、to pull latest stable repleased version phpredis ```https://github.com/phpredis/phpredis```
```
shell > cd /usr/local/
shell > wget https://github.com/phpredis/phpredis/archive/3.1.4.tar.gz
shell > cd phpredis-3.1.4 # 进入 phpredis 目录
shell > /usr/local/bin/phpize # php安装后的路径
shell > ./configure --with-php-config=/usr/local/bin/php-config
shell > make && make install
```
if you have question,like this
``` 
shell > /usr/local/bin/phpize 
return
Cannot find config.m4.
Make sure that you run '/usr/local/bin/phpize' in the top level source directory of the module
```
you need run command
```
cd /usr/loca/src
wget http://ftp.gnu.org/gnu/m4/m4-1.4.9.tar.gz
tar xf m4-1.4.9.tar.gz
mv m4-1.4.9 /usr/loca/
cd /usr/loca/m4-1.4.9
./configure && make && make install

cd /usr/loca/src
wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.62.tar.gz
tar xf autoconf-2.62.tar.gz
mv autoconf-2.62 /usr/loca/
cd /usr/loca/autoconf-2.62/
./configure && make && make install
```
2、phpredis can be used to store PHP sessions. To do this, configure session.save_handler and session.save_path in your php.ini to tell phpredis where to store the sessions:
```
shell > vim /usr/local/lib/php.ini
session.save_handler = redis
session.save_path = "tcp://host1:6379?weight=1, tcp://host2:6379?weight=2&timeout=2.5, tcp://host3:6379?weight=2&read_timeout=2.5"
```
3、update php.ini
```
shell > vim /usr/local/lib/php.ini
extension_dir = "/usr/local/lib/php/extensions/no-debug-non-zts-20170718/"
extension=redis.so
```
3、restart php
```
shell > ps aux | grep php-fpm
root 105292 0.0 0.0 145288 4984 ? Ss 10:58 0:00 php-fpm: master process (/usr/local/php/etc/php-fpm.conf)
nobody 105293 0.0 0.0 145352 6052 ? S 10:58 0:00 php-fpm: pool www
nobody 105294 0.0 0.0 145288 4632 ? S 10:58 0:00 php-fpm: pool www
root 105342 0.0 0.0 103344 860 pts/3 S+ 11:00 0:00 grep php-fpm
shell > kill 105292 # php-fpm master 进程
shell > /usr/local/bin/php-fpm
shell > ps aux | grep php-fpm
root 105347 0.0 0.0 147896 5140 ? Ss 11:00 0:00 php-fpm: master process (/usr/local/php/etc/php-fpm.conf)
nobody 105348 0.0 0.0 147896 4780 ? S 11:00 0:00 php-fpm: pool www
nobody 105349 0.0 0.0 147896 4780 ? S 11:00 0:00 php-fpm: pool www
root 105351 0.0 0.0 103344 864 pts/3 S+ 11:00 0:00 grep php-fpm
```
4、restart nginx
```
shell > /usr/local/nginx/sbin/nginx -s reload
```

5、check
```
shell > vim redis-demo.php
<?php
     //连接本地的 Redis 服务
     $redis = new Redis();
     $redis->connect('127.0.0.1', 6379);
     echo "Connection to server sucessfully"; //查看服务是否运行
     echo "Server is running: " . $redis->ping();
```

