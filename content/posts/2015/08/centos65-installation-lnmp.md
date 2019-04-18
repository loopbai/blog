---
title: 'CentOS6.5 安裝LNMP'
date: 2015-08-24T01:08:01+08:00
draft: false
---
#一、更新rpm
<a href="https://www.cnhzz.com/yum-repo/" target="_blank">參考連結</a>

**MYSQL源**
CentOS 6
`# yum install http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm`
CentOS 7
`# yum install http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm`

**NGINX源**
CentOS 6
`# yum install http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm`
CentOS 7
`# yum install http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm`

**EPEL源**
CentOS 6
`# yum install http://mirrors.ustc.edu.cn/fedora/epel/6/i386/epel-release-6-8.noarch.rpm`
CentOS 7
`# http://mirrors.yun-idc.com/epel/7/x86_64/e/epel-release-7-9.noarch.rpm`

**REMI源**
最新的PHP版本需要修改REMI.REPO中的ENABLED=0为1）# vi /etc/yum.repos.d/remi.repo
CentOS 6
`# yum install http://rpms.famillecollet.com/enterprise/remi-release-6.rpm`
CentOS 7
`# yum install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm`

**PUPPETLABS源**
CentOS 6
`# yum install http://yum.puppetlabs.com/puppetlabs-release-el-6.noarch.rpm`
CentOS 7
`# yum install http://yum.puppetlabs.com/puppetlabs-release-el-7.noarch.rpm`

**VARNISH源**
`# yum install https://repo.varnish-cache.org/redhat/varnish-4.0.el6.rpm`

#二、安裝Nginx
**安裝 NGINX 網頁伺服器**
`# yum install nginx`

#三、安裝PHP
>想install其它版本的PHP需要修改REMI.REPO中的ENABLED=0为1
>路徑：/etc/yum.repos.d/remi.repo
>[remi-php55]是PHP5.5
>[remi-php56]是PHP5.6
>選擇你要的版本做修改即可

`# vi /etc/yum.repos.d/remi.repo`

<img desc="" src="//imagehosting.rickyfun.net/201508/002.png">


**安裝 PHP**
`# yum install php`

**安裝其他PHP Lib**
`# yum install php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt libmcrypt-devel php-fpm`

#設定php, php-fpm
**設定php**
設定內容如下：
```config /etc/php.ini
cgi.fix_pathinfo=0
session.save_handler = files
session.save_path = "/var/lib/php/session"
date.timezone = "Asia/Taipei"
error_reporting = E_ALL & ~E_NOTICE
display_errors = On
magic_quotes_gpc = Off
```

**設定php session檔的權限**
`# mkdir /var/lib/php/session`
`# chown -R nginx:nginx /var/lib/php/session`

<img desc="" src="//imagehosting.rickyfun.net/201508/003.png">

**設定php-fpm**
>預設的 Pool 設定檔為 `/etc/php-fpm.d/www.conf` ，您可以適需要產生多組 Pool 來負責不同的網站服務。

```config /etc/php-fpm.d/www.conf
listen = 127.0.0.1:9000
listen.allowed_clients = 127.0.0.1
user = nginx
group = nginx
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
slowlog = /var/log/php-fpm/www-slow.log
php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
```

**設定 Log 檔的權限**
`# chown -R nginx:nginx /var/log/php-fpm`

#四、設定Nginx
>加入一個 index.php

```config /etc/nginx/conf.d/default.conf
    location / {
        root   /usr/share/nginx/html;
        index  index.php index.html index.htm;
    }
```

<img desc="" src="//imagehosting.rickyfun.net/201508/004.png">

>加入下面的 location 區塊設定，將 php 的請求使用 FastCGI 連結送給 PHP-FPM 處理。

```config /etc/nginx/conf.d/default.conf
location ~ \.php$ {
root /usr/share/nginx/html;
fastcgi_pass   127.0.0.1:9000;
fastcgi_index  index.php;
fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
include        fastcgi_params;
}
```

<img desc="" src="//imagehosting.rickyfun.net/201508/005.png">

#五、安裝MySQL
**安裝 MySQL**
`# yum install mysql mysql-server`

**啟動MySQL**
`# service mysqld start`

**配置 MySQL 基本設定**
`# /usr/bin/mysql_secure_installation`

#六、啟動服務
**啟動服務**
`# service nginx start`
`# service php-fpm start`

**設置開機啟動**
`# chkconfig --levels 235 nginx on`
`# chkconfig --levels 235 php-fpm on`
`# chkconfig --levels 235 mysqld on`