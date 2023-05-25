Домашнее задание "Размещаем свой RPM в своем репозитории"

1. Создаем виртуальную машину

- Создаем Vagrantfile с учетом требований;
- Включаем вм - vagrant up.

2. Создаем свой RPM пакет

- Подключаемся к вм - vagrant ssh, переходим в root - sudo -i

- Устанавливаем пакеты:
~~~
[root@repo ~]# sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
[root@repo ~]# sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
[root@repo ~]# yum install -y redhat-lsb-core wget rpmdevtools rpm-build createrepo yum-utils gcc
Failed to set locale, defaulting to C.UTF-8
CentOS Linux 8 - AppStream                                                                                                                                                                                   1.8 MB/s | 8.4 MB     00:04
CentOS Linux 8 - BaseOS                                                                                                                                                                                      2.6 MB/s | 4.6 MB     00:01

CentOS Linux 8 - Extras                                                                                                                                                                                       10 kB/s |  10 kB     00:00
Package yum-utils-4.0.17-5.el8.noarch is already installed.
Dependencies resolved.
Transaction Summary
=============================================================================================================================================================================================================================================
Install  104 Packages
Upgrade   28 Packages
~~~

- Возьмем пакет NGINX и соберем его с поддержкой openssl
- Загрузим SRPM пакет NGINX для дальнейшей работы над ним:
~~~
[root@repo ~]# wget https://nginx.org/packages/centos/8/SRPMS/nginx-1.20.2-1.el8.ngx.src.rpm
--2023-05-25 08:31:51--  https://nginx.org/packages/centos/8/SRPMS/nginx-1.20.2-1.el8.ngx.src.rpm
Resolving nginx.org (nginx.org)... 3.125.197.172, 52.58.199.22, 2a05:d014:edb:5702::6, ...
Connecting to nginx.org (nginx.org)|3.125.197.172|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1086865 (1.0M) [application/x-redhat-package-manager]
Saving to: 'nginx-1.20.2-1.el8.ngx.src.rpm'

nginx-1.20.2-1.el8.ngx.src.rpm                              100%[========================================================================================================================================>]   1.04M  3.10MB/s    in 0.3s

2023-05-25 08:31:52 (3.10 MB/s) - 'nginx-1.20.2-1.el8.ngx.src.rpm' saved [1086865/1086865]
~~~

- Установим и создадим древо каталогов для сборки:
~~~
[root@repo ~]# rpm -i nginx-1.*
warning: nginx-1.20.2-1.el8.ngx.src.rpm: Header V4 RSA/SHA1 Signature, key ID 7bd9bf62: NOKEY
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
warning: user builder does not exist - using root
warning: group builder does not exist - using root
[root@repo ~]# ls
anaconda-ks.cfg  nginx-1.20.2-1.el8.ngx.src.rpm  original-ks.cfg  rpmbuild
~~~

- Cкачаем и разархивируем исходник для openssl:
~~~
[root@repo ~]# wget https://github.com/openssl/openssl/archive/refs/heads/OpenSSL_1_1_1-stable.zip
--2023-05-25 08:41:01--  https://github.com/openssl/openssl/archive/refs/heads/OpenSSL_1_1_1-stable.zip
Resolving github.com (github.com)... 140.82.121.3
Connecting to github.com (github.com)|140.82.121.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://codeload.github.com/openssl/openssl/zip/refs/heads/OpenSSL_1_1_1-stable [following]
--2023-05-25 08:41:02--  https://codeload.github.com/openssl/openssl/zip/refs/heads/OpenSSL_1_1_1-stable
Resolving codeload.github.com (codeload.github.com)... 140.82.121.9
Connecting to codeload.github.com (codeload.github.com)|140.82.121.9|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 11922688 (11M) [application/zip]
Saving to: 'OpenSSL_1_1_1-stable.zip'

OpenSSL_1_1_1-stable.zip                                    100%[========================================================================================================================================>]  11.37M  5.24MB/s    in 2.2s

2023-05-25 08:41:04 (5.24 MB/s) - 'OpenSSL_1_1_1-stable.zip' saved [11922688/11922688]
[root@repo ~]# unzip OpenSSL_1_1_1-stable.zip
~~~

- Cтавим все зависимости:
~~~
[root@repo ~]# yum-builddep rpmbuild/SPECS/nginx.spec
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 0:27:19 ago on Thu May 25 08:36:24 2023.
Package systemd-239-41.el8_3.x86_64 is already installed.
Dependencies resolved.
~~~

- Редактируем spec файл, чтобы NGINX собрался с необходимыми опциями:
~~~
[root@repo ~]# nano rpmbuild/SPECS/nginx.spec

%build
./configure %{BASE_CONFIGURE_ARGS} \
    --with-cc-opt="%{WITH_CC_OPT}" \
    --with-ld-opt="%{WITH_LD_OPT}" \
    --with-openssl=/root/openssl-OpenSSL_1_1_1-stable
~~~

- Сборка RPM пакета:
~~~
[root@repo ~]# rpmbuild -bb rpmbuild/SPECS/nginx.spec

Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.7djqH4
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd nginx-1.20.2
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/nginx-1.20.2-1.el8.ngx.x86_64
+ exit 0
~~~

- Убедимся, что RPM пакеты собрались:
~~~
[root@repo ~]# ll rpmbuild/RPMS/x86_64/
total 4456
-rw-r--r--. 1 root root 2061432 May 25 09:55 nginx-1.20.2-1.el8.ngx.x86_64.rpm
-rw-r--r--. 1 root root 2494836 May 25 09:55 nginx-debuginfo-1.20.2-1.el8.ngx.x86_64.rpm
~~~

- Установим пакет и убедимся, что nginx работает:
~~~
[root@repo ~]# yum localinstall -y rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el8.ngx.x86_64.rpm

  Verifying        : nginx-1:1.20.2-1.el8.ngx.x86_64                                                                                                                                                                                     1/1

Installed:
  nginx-1:1.20.2-1.el8.ngx.x86_64

[root@repo ~]# systemctl start nginx
[root@repo ~]# systemctl status nginx
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2023-05-25 10:03:06 UTC; 5s ago
     Docs: http://nginx.org/en/docs/
  Process: 47797 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
 Main PID: 47798 (nginx)
    Tasks: 2 (limit: 1133)
   Memory: 23.7M
   CGroup: /system.slice/nginx.service
           ├─47798 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
           └─47799 nginx: worker process

May 25 10:03:06 repo systemd[1]: Starting nginx - high performance web server...
May 25 10:03:06 repo systemd[1]: nginx.service: Can't open PID file /var/run/nginx.pid (yet?) after start: No such file or directory
May 25 10:03:06 repo systemd[1]: Started nginx - high performance web server.
~~~

3. Создадим свой репозиторий и разместим там ранее собранный RPM

- Создадим каталог для репозитория, копируем собранный RPM и RPM для установки репозитория Persona-Server:
~~~
[root@repo ~]# mkdir /usr/share/nginx/html/repo
[root@repo ~]# cp rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el8.ngx.x86_64.rpm /usr/share/nginx/html/repo/
[root@repo ~]# ls /usr/share/nginx/html/repo/
nginx-1.20.2-1.el8.ngx.x86_64.rpm

[root@repo ~]# wget https://downloads.percona.com/downloads/percona-distribution-mysql-ps/percona-distribution-mysql-ps-8.0.28/binary/redhat/8/x86_64/percona-orchestrator-3.2.6-2.el8.x86_64.rpm -O /usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm
--2023-05-25 10:25:05--  https://downloads.percona.com/downloads/percona-distribution-mysql-ps/percona-distribution-mysql-ps-8.0.28/binary/redhat/8/x86_64/percona-orchestrator-3.2.6-2.el8.x86_64.rpm
Resolving downloads.percona.com (downloads.percona.com)... 74.121.199.231, 162.220.4.221, 162.220.4.222
Connecting to downloads.percona.com (downloads.percona.com)|74.121.199.231|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5222976 (5.0M) [application/octet-stream]
Saving to: '/usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm'

/usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8 100%[========================================================================================================================================>]   4.98M  3.77MB/s    in 1.3s

2023-05-25 10:25:07 (3.77 MB/s) - '/usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm' saved [5222976/5222976]

[root@repo ~]# ls /usr/share/nginx/html/repo/
nginx-1.20.2-1.el8.ngx.x86_64.rpm  percona-orchestrator-3.2.6-2.el8.x86_64.rpm
~~~

- Инициализируем репозиторий командой:
~~~
[root@repo ~]# createrepo /usr/share/nginx/html/repo/
Directory walk started
Directory walk done - 2 packages
Temporary output repo path: /usr/share/nginx/html/repo/.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished
~~~

- Для прозрачности настроим в NGINX доступ к листингу каталога (добавим директиву autoindex on):
~~~
[root@repo ~]# nano  /etc/nginx/conf.d/default.conf
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        autoindex on;
    }
~~~

- Проверяем синтаксис и перезапускаем NGINX:
~~~
[root@repo ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@repo ~]# nginx -s reload
~~~

- Проверям через curl:
~~~
[root@repo ~]# curl -a http://localhost/repo/
<html>
<head><title>Index of /repo/</title></head>
<body>
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
<a href="repodata/">repodata/</a>                                          25-May-2023 10:29                   -
<a href="nginx-1.20.2-1.el8.ngx.x86_64.rpm">nginx-1.20.2-1.el8.ngx.x86_64.rpm</a>                  25-May-2023 10:23             2061432
<a href="percona-orchestrator-3.2.6-2.el8.x86_64.rpm">percona-orchestrator-3.2.6-2.el8.x86_64.rpm</a>        16-Feb-2022 15:57             5222976
</pre><hr></body>
</html>
~~~

- Добавляем репозиторий в /etc/yum.repos.d:
~~~
[root@repo ~]# cat >> /etc/yum.repos.d/otus.repo << EOF
> [otus]
> name=otus-linux
> baseurl=http://localhost/repo
> gpgcheck=0
> enabled=1
> EOF
[root@repo ~]# cat /etc/yum.repos.d/otus.repo
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
~~~

- Убедимся, что репозиторий подключился и посмотрим, что в нем есть:
~~~
[root@repo ~]# yum repolist enabled | grep otus
Failed to set locale, defaulting to C.UTF-8
otus                            otus-linux

[root@repo ~]# yum list | grep otus
Failed to set locale, defaulting to C.UTF-8
otus-linux                                       15 kB/s | 2.8 kB     00:00
percona-orchestrator.x86_64                            2:3.2.6-2.el8                                          otus
~~~

- Установим репозиторий percona-release:
~~~
[root@repo ~]# yum install percona-orchestrator.x86_64 -y

Installed:
  jq-1.5-12.el8.x86_64                                                 oniguruma-6.8.2-2.el8.x86_64                                                 percona-orchestrator-2:3.2.6-2.el8.x86_64

Complete!
~~~



