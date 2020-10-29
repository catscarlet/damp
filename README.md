# DAMP

Docker + Apache + MySQL + PHP. Very Moist!

[![996.icu](https://img.shields.io/badge/link-996.icu-red.svg)](https://996.icu)

DAMP is a composing tool that will help you deploy Apache, MySQL and PHP in one shot.

DAMP is named to pay homage to LAMP.

The idea is to deploy the server environment as same as LAMP on a brand new server. *(If you want to replace an exist server, please wait for my another implementation)*

## Features

- Easy to deploy
- Support HTTP/2 and TLS1.3

------

## Introduction

*Before read, please ensure that you have known about apache2 and docker*

### Description

- Apache version 2.4, works in event mode.
- PHP version 7.4, works by PHP-FPM
- MySQL version 5.7.

项目实现可以参考每个组件的 Dockerfile。所有组件均未指定小版本号，如有需要请手动指定。

Refer to Dockerfiles for more information. All X.Y.Z version number Z are not assigned. Assign by yourself if you wish.

### Structure

```
.
├── apache2-event
│   ├── Dockerfile
│   └── usr
│       └── local
│           └── apache2
│               └── conf
│                   ├── extra
│                   │   ├── httpd-ssl.conf
│                   │   └── httpd-ssl.conf_bak
│                   ├── httpd.conf
│                   └── httpd.conf_bak
├── docker-compose.yaml
├── logs
│   └── apache2
│       └── README.md
├── mysql-db-storage
├── php-fpm
│   ├── Dockerfile
│   └── etc
│       └── apt
│           └── sources.list
├── README.md
├── README_zh-cmn-Hans.md
└── sites
    ├── sites-conf
    │   ├── 000-default
    │   │   ├── 000-default.conf
    │   │   ├── vhost1.crt
    │   │   └── vhost1.key
    │   └── 001-damp.test
    │       ├── 001-damp.test.conf
    │       ├── damp.test-key.pem
    │       └── damp.test.pem
    └── sites-document
        ├── 000-default
        │   └── index.html
        └── 001-damp.test
            ├── phpinfo.php
            └── php-www-data-test.php
```

Among them:

The apache2-event directory includes all configurations about Apache, including a default configuration file backup. You can compare them to know what is changed in this project. Also you can change the config as you wish.

The php-fpm directory includes all configurations about php. The Dockerfile includes how damp_php-fpm compiled。Also you can change the config as you wish.*（If you need to use damp in China mainland, uncomment `COPY etc/ /etc/` in Dockerfile, and it will use USTC open source software mirror to speed compiling, instead of using deb.debian.org*

The mysql-db-storage directory stores MySQL Database files (as `/var/lib/mysql/` in traditional ways)**(Notice, this directory is created after damp runs once. Permissions is controlled by MySQL Docker)**

The logs directory stores apache2 logs (as `/var/log/apache2/` in traditional ways)

The sites directory stores websites files:

- sites-conf directories: store website setting. (as `apache2/sites-conf/` in traditional ways)
- sites-document directories: store website files. (as `/var/www/html/` in traditional ways)

------

## Usage

### Demo

In sites directory, there are a default site(IP direct access), and a demo site(damp.test).

The demo site's domain is damp.test。You can test by editing your hosts。As example if you want to deploy and test on same local host, add this line in hosts:

```
127.0.0.1	damp.test
```

And visit http://damp.test/ . DAMP also support HTTPS. Visit https://damp.test/ for sure (and of course a self-signed warning will pop up)

### Add site

(Skip this part if you just want to try the demo)

*Suggest you to do this by doing copy-paste the damp.test demo*

1. Create relevant directory in sites-conf. Put relevant conf, certificate and key.
2. Create relevant directory in sites-document. Put your site files in it.
3. Exec `docker-compose up -d` to start.

### Redeploy

After adding or changing configurations, the easiest way to redeploy is run:

```
docker-compose down && docker-compose up --force-recreate --build -d
```

If you want to upgrade Apache / MySQL / PHP, which needs to rebuild images, you need to remove all exist damp images and it takes time.

Please backup all your data before doing upgrade

```
docker-compose down --rmi all && docker-compose up --force-recreate --build -d
```

For more information, please refer to Docker and Docker Compose Documents.

------

## Something notable things.

First thing First: **If you want to use DAMP in production environment, please change the default password of MySQL, the `MYSQL_ROOT_PASSWORD` in `docker-compose.yaml`, then remove all `_EXAMPLE` parameters**

1. About the logs of Apache, HTTP log and HTTPS log are mixed now. This is a Apache default site-conf thing. Maybe we should change it.
2. Also about logs, the Apache log files are output to logs/apache/, while the logs of php-fpm and mysql are output to stdout of docker, that leads to `docker-compose logs`
3. In docker-compose, the Timezone setting (environment TZ) only affect logs of apache and php. [PHP: The default timezone used is UTC if you don't set it][1]
4. MySQL won't get affected by system timezone setting, even the log. [MySQL's default time zone of timestamps in messages written to the error log value is UTC][2]
5. You can't access MySQL's port 3306 outside of DAMP by default. To make it accessible, uncomment the MySQL ports parameters in `docker-compose.yaml`

## Todo list

- MySQL Upgrade to 8.0
- Separate Apache's HTTP log and HTTPS log in demo
- HTTP/3 (If Apache starts to support it, damp would follow)

------

## Reference

[1]: https://www.php.net/manual/en/function.date-default-timezone-get.php
[2]: https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_log_timestamps
