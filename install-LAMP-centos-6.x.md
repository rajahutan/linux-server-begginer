# Setup New Server (NGINX,MYSQL,PHP) CENTOS 6.x
In this totorial we use
- NGINX version 1.12
- PHP version 7
- MySql version 5.7
- Centos 6.9

## Setup Time
1. Remove old locatime setup:
    > rm /etc/localtime

2. Check your timezone on `/usr/share/zoneinfo/`
3. Make Symbolyc link to selected timezone file
    For Indonesian WITA / Ujung pandang /  Makasar
    >ln -s  /usr/share/zoneinfo/Asia/Ujung_Pandang localtime

    For Indonesian WIB / Jakarta
    >ln -s  /usr/share/zoneinfo/Asia/Jakarta localtime
4. Update system clock to timeserver.  
    >ntpdate id.pool.ntp.org

### Issue
- No Ntp server setting.  
    install ntp server.  
    We will release next totorial for each issue.

## Install Mysql
### Install Mysql
1. Install Version 5.7 using rpm release 5.7
    >yum localinstall https://dev.mysql.com/get/mysql57-community-release-el6-9.noarch.rpm
    >yum install mysql-community-server
2. Start mysqld
    mysql service named as mysql or mysqld
    >service mysqld start 
    use restart after update
    or
    >/etc/init.d/mysqld start ## use restart after update
3. Run On StartUp
    >chkconfig --levels 235 mysqld on
4. Get Your Generated Random root Password
    >grep 'A temporary password is generated for root@localhost' /var/log/mysqld.log |tail -1
    Used on step 5 securing instalation login for first time
5. Securing Mysql Instalation
    >/usr/bin/mysql_secure_installation
    The easy step is answer all with yes.  
    If you don’t want some reason, do a “MySQL Secure Installation” then at least it’s very important to change the root user’s password
    
### Setup Remote Access

### Manage Mysql
1. Try to Login to mysql
    >mysqladmin -u root -p [your_password_prompted]
2. Create user and Database
    ```mysql
    create database [your_db_name];
    create user [your_user_name]@localhost identified by '[your_password]';
    grant all privileges on [your_db_name].* to [your_user_name]@localhost identified by '[your_password]';
    flush privileges;
    ```mysql

### Issue
- Remote mysql fail  
Check Firewal and localhost listener and host file.  
We will release next totorial for each issue  

## NGINX
1. Make new config file on yum repo for nginx.
    >vim /etc/yum.repos.d/nginx.repo
2. Add config to of file
    ```txt
    [nginx]
    name=nginx repo
    baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
    gpgcheck=0
    enabled=1
    ```
3. Execute Command
For Install
    >sudo yum install nginx
For Update
    >sudo yum update nginx
4. Run Service
    >sudo service nginx stop
5. Run Nginx On startup
    >chkconfig nginx on

## PHP 7

### (1) Prepare Instalation
Update Yum
    >yum install epel-release
    >rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
    >rpm -Uvh https://mirror.webtatic.com/yum/el6/latest.rpm
    >rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
Install PHP 7 and modeule
    >yum -y install php70w-fpm php70w-cli php70w-gd php70w-mcrypt php70w-mysql php70w-pear php70w-xml php70w-mbstring php70w-pdo php70w-json php70w-pecl-apcu php70w-pecl-apcu-devel
Finally, check the PHP version from server terminal to verify that PHP installed correctly.
    >php -v

### (2) Configure PHP7-FPM
1. Edit php-fpm config. location mybe different.
2. Make sure location of config php-fpm
    >where is fpm
3. Edit conf file
    >vim /etc/php-fpm.d/www.conf
4. Find line using vim command `:/[text-to-search]`
5. Edit this line
    ```txt
    user = nginx
    group = nginx
    listen.owner = nginx
    listen.group = nginx
    listen.mode = 0660
    ```
6. Uncomment or cofig listerner unix socket. This option the most often cause 502 bad gateaway nginx-php.
    >listen = /var/run/php-fpm/php-fpm.sock
7. Make sure /etc/nginx/fastcgi_params looks like this:
    ```txt
    fastcgi_param  QUERY_STRING       $query_string;
    fastcgi_param  REQUEST_METHOD     $request_method;
    fastcgi_param  CONTENT_TYPE       $content_type;
    fastcgi_param  CONTENT_LENGTH     $content_length;

    fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
    fastcgi_param  REQUEST_URI        $request_uri;
    fastcgi_param  DOCUMENT_URI       $document_uri;
    fastcgi_param  DOCUMENT_ROOT      $document_root;
    fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param  SERVER_PROTOCOL    $server_protocol;
    fastcgi_param  PATH_INFO          $fastcgi_script_name;
    fastcgi_param  HTTPS              $https if_not_empty;

    fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
    fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

    fastcgi_param  REMOTE_ADDR        $remote_addr;
    fastcgi_param  REMOTE_PORT        $remote_port;
    fastcgi_param  SERVER_ADDR        $server_addr;
    fastcgi_param  SERVER_PORT        $server_port;
    fastcgi_param  SERVER_NAME        $server_name;
    ```
8. PHP only, required if PHP was built with --enable-force-cgi-redirect
    >fastcgi_param  REDIRECT_STATUS    200;
9. These two lines somotimes missing from /etc/nginx/fastcgi_params, make sure they are there!.This often make file not found on php execution
    >fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    >fastcgi_param  PATH_INFO          $fastcgi_script_name;
10. Run php-fpm service
    >service php-fpm start
11. Did't Works? Dont panix. Go to Issue section below

### (3) MySQL Support In PHP5

#### Issue
1. 502 bad gateaway nginx-php
Make sure nginx listener port or socket config is ok.  
By default PHP-FPM is listening on port 9000 on 127.0.0.1. It is also possible to make PHP-FPM use a Unix socket which avoids the TCP overhead.
Check if php-fpm.sock created under listen config directory.  
Check step 6  
2. File Not Found on php script
Fastcgi_param config problem. Check step 7 and 8.  
Is problem still exist. Try to manual setup on location directive to path of your server ducument root.  
Edit nginx default.conf
    >fastcgi_param SCRIPT_FILENAME /usr/share/nginx/html
    ```nginx
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        include        fastcgi_params;
    }
    ```
3. php-fpm: unrecognized service
Find the location of fpm-service depend on your php version.  
In this tutorial the folder named php-fpm. Sometimes the folder or service named by version. Like php5-fpm.
    >whereis fpm
Then execute the right service  
We will release next totorial for each issue
