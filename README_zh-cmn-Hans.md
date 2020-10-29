# DAMP

Docker + Apache + MySQL + PHP，特别潮的解决方案。

[![996.icu](https://img.shields.io/badge/link-996.icu-red.svg)](https://996.icu) [![LICENSE](https://img.shields.io/badge/license-Anti%20996-blue.svg)](https://github.com/996icu/996.ICU/blob/master/LICENSE)

DAMP 是一个基于 Docker 的用于搭建 Apache + MySQL + PHP 服务器环境的快速编排工具。

DAMP 取名致敬于 LAMP。

项目基于 Docker 和 Docker Compose 实现。

本项目的初衷是为了在全新环境下快速建立与 LAMP 相同的网站服务器环境。*（如果您是希望替换现有的网站项目，请参考这个项目：[DAMP Alternative](https://github.com/catscarlet/damp-alternative)）*

DAMP 项目使用 Mozilla Public License Version 2.0 和 Anti 996 License Version 1.0 (Draft) 授权

## 特性

- 方便的部署方式
- 支持 HTTP/2 和 TLS1.3

------

## 简介

*阅读前请确认您对 apache2 和 docker 有一定的了解。*

### 简述

项目使用 apache2 和 php 实现 DAMP 中的 A 和 P， MySQL Community Server 实现 DAMP 中的 M。

- apache2 使用 2.4 版本，工作于 event 模式。
- php 使用 7.4 版本，使用 php-fpm 实现。
- MySQL 使用 5.7 版本。（非 8.0）

项目实现可以参考每个组件的 Dockerfile。所有组件均未指定小版本号，如有需要请手动指定。

### 项目结构

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

其中：

apache2-event 目录下存放了所有与 apache2 相关的定义配置，并包含一份默认配置备份，你可以对比两个文件的区别并了解本项目针对默认设置都有哪些改动。您也可以根据自己的需要更改这些配置。

php-fpm 目录下存放了所有与 php 相关的定义配置，其中 Dockerfile 中包含了 php 相关模块的编译。您也可以根据自己的需要更改这些配置。*（如果您是中国大陆用户的话，可以编辑 Dockerfile 取消对 `COPY etc/ /etc/` 的注释，这样可以将编译库文件的源替换成中国科技大学的镜像源以提供下载速度）*

mysql-db-storage 目录下存放 MySQL 数据库文件（相当于 `/var/lib/mysql/`）。**（注意，此目录是在启用 damp 之后创建的，由 MySQL Docker 维护用户权限）**

logs 目录下存放 apache2 的日志（相当于 `/var/log/apache2/`）

sites 目录下存放网站配置，其中：

- sites-conf：存放每个网站的配置文件（相当于 `apache2/sites-conf/`）
- sites-document：存放每个网站的应用文件（相当于 `/var/www/html/`）

------

## 使用

### Demo

sites 目录下存放了一份默认网站（IP直接访问时的网站），和一份样例网站。

样例网站域名为 damp.test。您可以编辑客户端的 hosts 用于测试。比如如果您打算在本机测试的话，在 hosts 中添加

```
127.0.0.1	damp.test
```

并访问 http://damp.test/ 即可。DAMP 同样支持 HTTPS 访问，您可以直接访问 https://damp.test/ 进行访问（当然会有 self-signed 警示）

### 添加自己的网站

(如果您只是想测试下 Demo 的话，可以跳过此小结)

*建议基于样例网站进行复制粘贴式的配置*

1. 在 sites-conf 下建立相应的目录，并添加配置文件、证书、私钥
2. 在 sites-document 下建立相应的目录，并添加网站文件
3. 执行 `docker-compose up -d` 启动。

### 重新部署

当你添加或更改了配置文件后，最简单的办法是执行以下命令，即可直接重建所有容器数据。

```
docker-compose down && docker-compose up --force-recreate --build -d
```

如果需要升级 Apache / MySQL / PHP 的话，需要删除所有已生成的镜像，并手动更新所有 damp 的依赖镜像。这样做会重新编译 PHP，需要消耗一定的时间。

升级基本服务请酌情，考虑你的现有代码是否能够稳定兼容新版本，并强烈建议升级前手动备份所有数据。

```
docker-compose down --rmi all && docker-compose up --force-recreate --build -d
```

关于 Docker 的使用，请参考 Docker 和 Docker Compose 的文档。

------

## 一些事项

首先：**如果要在生产环境下使用本项目，请修改 docker-compose.yaml 中 MySQL 的默认密码 `MYSQL_ROOT_PASSWORD`，并删除所有 `_EXAMPLE` 参数！**

1. Apache 的日志，HTTP 和 HTTPS 日志目前写在了一起。
2. Apache 的日志输出到了 logs/apache/ 下，而 php-fpm 和 mysql 的日志输出到了 Docker 的标准输出中，也就是需要用 `docker-compose logs` 进行查看
3. docker-compose 中的时区设定（environment TZ）只负责 apache 和 php 的日志记录。[修改系统的时区不会影响网站应用的时区][1]
4. MySQL 对系统时区不感冒，[即使只是修改错误日志的时区也需要修改数据表][2]
5. 默认无法从 DAMP 外部访问 MySQL 的 3306 端口。如有需要，请取消 docker-compose.yaml 中 MySQL 的 ports 参数注释。

## Todo list

- MySQL 升级到 8.0
- 分离 Apache 的 HTTP 和 HTTPS 日志
- HTTP/3 (If Apache starts to support it, damp will follow)

------

## Reference

[1]: https://www.php.net/manual/en/function.date-default-timezone-get.php
[2]: https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_log_timestamps
