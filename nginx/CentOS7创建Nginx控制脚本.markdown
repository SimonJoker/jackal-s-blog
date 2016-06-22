# CentOS7 创建 nginx控制脚本
**摘要:**
```
nginx 安装完成后我们需要创建启动、关闭、重启nginx 服务的控制脚本，除了传统的方法发来创建启动脚本外，还可以通过 RHEL 7新的特性，自定义系统服务的方法来创建Nginx 控制脚本

```
## 一、准备内容（centos systemd 添加自定义系统服务）
### 1、systemd：
CentOS 7的服务systemctl脚本存放在：`/usr/lib/systemd/`，有系统（system）和用户（user）之分，即：`/usr/lib/systemd/system` ，`/usr/lib/systemd/user`；每个服务区以一个`.service`结尾，一般会分为3部分：`[Unit]、[Service]和[Install]`
### 2、创建service:
就以nginx为例,在`/usr/lib/systemd/system`下创建nginx.service文件内容如下（看应用需求也可以在 ／usr/lib/systemd/usr下创建）：
```
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
#### [Unit]：
Description : 服务的简单描述
Documentation ： 服务文档
After= : 依赖，仅当依赖的服务启动之后再启动自定义的服务单元
#### [Service]：
- Type : 启动类型simple、forking、oneshot、notify、dbus
>Type=simple（默认值）：systemd认为该服务将立即启动。服务进程不会fork。如果该服务要启动其他服务，不要使用此类型启动，除非该服务是socket激活型。 Type=forking：systemd认为当该服务进程fork，且父进程退出后服务启动成功。对于常规的守护进程（daemon），除非你确定此启动方式无法满足需求，使用此类型启动即可。使用此启动类型应同时指定 PIDFile=，以便systemd能够跟踪服务的主进程。 Type=oneshot：这一选项适用于只执行一项任务、随后立即退出的服务。可能需要同时设置 RemainAfterExit=yes 使得 systemd 在服务进程退出之后仍然认为服务处于激活状态。 Type=notify：与 Type=simple 相同，但约定服务会在就绪后向 systemd 发送一个信号。这一通知的实现由 libsystemd-daemon.so 提供。 Type=dbus：若以此方式启动，当指定的 BusName 出现在DBus系统总线上时，systemd认为服务就绪。

- PIDFile ： pid文件路径
- ExecStartPre ：启动前要做什么，上文中是测试配置文件 －t  
- ExecStart：启动
- ExecReload：重载
- ExecStop：停止
- PrivateTmp：True表示给服务分配独立的临时空间

#### [Install]:
WantedBy：服务安装的用户模式，从字面上看，就是想要使用这个服务的有是谁？上文中使用的是：multi-user.target ，就是指想要使用这个服务的目录是多用户。「以上全是个人理解，瞎猜的，如有不当，请大家多多指教」每一个.target实际上是链接到我们单位文件的集合,当我们执行：
```
#开机启动 nginx.service
sudo systemctl enable nginx.service

#服务不开机启动
systemctl disable nginx.service

#查看服务是否是开机启动
systemctl is-enabled nginx.service

```
就会在/etc/systemd/system/multi-user.target.wants/目录下新建一个/usr/lib/systemd/system/nginx.service 文件的链接。

### 3、操作Service:
```
#启动服务
$ sudo systemctl start nginx.service

#查看日志
$ sudo journalctl -f -u nginx.service
-- Logs begin at 四 2015-06-25 17:32:20 CST. --
6月 25 10:28:24 Leco.lan systemd[1]: Starting nginx - high performance web server...
6月 25 10:28:24 Leco.lan nginx[7976]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
6月 25 10:28:24 Leco.lan nginx[7976]: nginx: configuration file /etc/nginx/nginx.conf test is successful
6月 25 10:28:24 Leco.lan systemd[1]: Started nginx - high performance web server.

#重启
$ sudo systemctl restart nginx.service

#重载
$ sudo systemctl reload nginx.service

#停止
$ sudo systemctl stop nginx.service
```
## 二、编写nginx.service
```
vi /usr/lib/systemd/system/nginx.service

#输入内容
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target

```
修改文件权限：
```
sudo chmod +x /usr/lib/systemd/system/nginx.service

sudo systemctl enable nginx.service

```

nginx 服务控制命令：
```
sudo systemctl start nginx.service
sudo systemctl reload nginx.service
sudo systemctl restart nginx.service
sudo systemctl stop nginx.service

```
实时查看日志：
```
$ journalctl -f -u nginx.service
```
### 注意上面这几个路径：
```
#下面这几个路径是你的nginx安装的目录，务必不要弄错。
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
```

## 三、传统启动nginx 脚本方法
### 1、创建脚本
```
vim /etc/init.d/nginx

```
在网上直接搜索现成的启动脚本（相信我有很多）
```
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemin
#
# chkconfig:   - 85 15
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# pidfile:     /run/nginx/nginx.pid
# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/sbin/nginx"

prog=$(basename $nginx)

NGINX_CONF_FILE="/etc/nginx/nginx.conf"

lockfile=/var/lock/nginx.lock

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```
注意几个地方的配置，就是上在nginx中编译时设置的那些目录：
```
# config: /etc/nginx/nginx.conf
# pidfile: /run/nginx/nginx.pid
nginx="/usr/sbin/nginx
NGINX_CONF_FILE="/etc/nginx/nginx.conf
lockfile=/var/lock/nginx.lock
```

### 2、添加服务
```
chmod a+x /etc/init.d/nginx

chkconfig --add nginx

chkconfig --list nginx
nginx          	0:off	1:off	2:off	3:off	4:off	5:off	6:off
```
上面其实就是在／etc/rc.d/rc5.d/目录下创建了一个链接。如下：
```
$ cd /etc/rc.d/rc5.d/

$ ll |grep nginx
lrwxrwxrwx. 1 root root 15 6月  24 16:14 K15nginx -> ../init.d/nginx
```

### 3、使用
```
$ service nginx start  
$ service nginx stop  
$ service nginx restart  
$ service nginx reload  

$ /etc/init.d/nginx start  
$ /etc/init.d/nginx stop  
$ /etc/init.d/nginx restart  
$ /etc/init.d/nginx reload
```
