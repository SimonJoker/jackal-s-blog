
# Mysql5.7 源码编译安装
## mysql5.7主要特性
- 更好的性能：对于多核CPU、固态硬盘、锁有着更好的优化，美妙100W QPS已不再是mysql的追求，也许在下个版本上200W QPS才是我们所期待的
- 更好的 INNODB 存储引擎
- 更为健壮的复制功能： 复制带来了数据完全不丢失的方案，传统金融用户也可以选择使用 Mysql 数据库，此外，GTID在线平滑升级也变为可能
- 更好的优化器： 优化器代码重构意义将在这个版本以及以后的版本中带来巨大的改进，oracle 官方正在解决mysql之前最大的难题
- 原声 json 类型的支持
- 更好的地理信息服务支持：InnoDB 原声支持地理位置类型，支持GeoJSON, GeoHash 特性
- 新增sys库：以后这回事DBA访问最频繁的库
- MySQL 5.7已经作为数据库可选项添加到《OneinStack》

## 安装mysql5.7
### 开始安装
- 系统：CentOS7

#### 1、安装编译环境依赖包
```
yum -y install gcc gcc-c++ ncurses ncurses-devel cmake
```
#### 2、下载数据库源码包
```
#官网中只需要下载这一个源码包即可，这个包包含了 boost 的 mysql 源码
wget http://downloads.mysql.com/archives/get/file/mysql-boost-5.7.12.tar.gz
wget http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.13.tar.gz
```
### 3、添加 mysql 用户
```
useradd -M -s /sbin/nologin mysql
```
#### 4、编译安装源码
```
cd /usr/local/src/mysql/mysql-5.7.12

# 预编译

cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
 -DMYSQL_DATADIR=/data/mysql \
 -DDOWNLOAD_BOOST=1 \
 -DWITH_BOOST=./boost \
 -DSYSCONFDIR=/etc \
 -DWITH_INNOBASE_STORAGE_ENGINE=1 \
 -DWITH_PARTITION_STORAGE_ENGINE=1 \
 -DWITH_FEDERATED_STORAGE_ENGINE=1 \
 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
 -DWITH_MYISAM_STORAGE_ENGINE=1 \
 -DENABLED_LOCAL_INFILE=1 \
 -DENABLE_DTRACE=0 \
 -DDEFAULT_CHARSET=utf8mb4 \
 -DDEFAULT_COLLATION=utf8mb4_general_ci \
 -DWITH_EMBEDDED_SERVER=1


# 编译&安装（编译过程很消耗资源，内存过小编译可能不能成功，将话费大量时间）

make -j `grep processor /proc/cpuinfo | wc -l`
make install
```

#### 5、mysql 启动设置
```
# 复制服务启动脚本 & 使其可运行
/bin/cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld

# 设置开机启动
chkconfig --add mysqld
chkconfig mysqld on

```
#### 6、配置数据
##### /etc/my.cnf 可供参考配置文件内容
```
[client]
port = 3306
socket = /tmp/mysql.sock
default-character-set = utf8mb4

[mysqld]
port = 3306
socket = /tmp/mysql.sock

basedir = /usr/local/mysql
datadir = /data/mysql
pid-file = /data/mysql/mysql.pid
user = mysql
bind-address = 0.0.0.0
server-id = 1

init-connect = 'SET NAMES utf8mb4'
character-set-server = utf8mb4

#skip-name-resolve
#skip-networking
back_log = 300

max_connections = 1000
max_connect_errors = 6000
open_files_limit = 65535
table_open_cache = 128
max_allowed_packet = 4M
binlog_cache_size = 1M
max_heap_table_size = 8M
tmp_table_size = 4M

read_buffer_size = 2M
read_rnd_buffer_size = 8M
sort_buffer_size = 8M
join_buffer_size = 8M
key_buffer_size = 4M

thread_cache_size = 8

query_cache_type = 1
query_cache_size = 8M
query_cache_limit = 2M

ft_min_word_len = 4

log_bin = mysql-bin
binlog_format = mixed
expire_logs_days = 30
log_error = /data/mysql/mysql_error.log
slow_query_log = 1
long_query_time = 1
slow_query_log_file = /data/mysql/mysql-slow.log

performance_schema = 0

explicit_defaults_for_timestamp

#lower_case_table_names = 1

skip-external-locking

default_storage_engine = InnoDB
#default_storage_engine = MyISAM
innodb_file_per_table = 1
innodb_open_files = 500
innodb_buffer_pool_size = 64M
innodb_write_io_threads = 4
innodb_read_io_threads = 4
innodb_thread_concurrency = 0
innodb_purge_threads =1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 32M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120

bulk_insert_buffer_size = 8M
myisam_sort_buffer_size = 8M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1

interactive_timeout = 28800
wait_timeout = 28800

[mysqldump]
quick
max_allowed_packet = 16M

[myisamchk]
key_buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M

```
## 初始化数据库
```
/usr/local/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql
```
###### 注：之前版本mysql_install_db是在mysql_basedir/script下，5.7放在了mysql_install_db/bin目录下,且已被废弃“–initialize”会生成一个随机密(~/.mysql_secret)，而”–initialize-insecure”不会生成密码–datadir目标目录下不能有数据文件

## 启动数据库
```
service mysqld start
```

## 设置数据库用户密码 & 权限
```
dbrootpwd=oneinstack  #数据库root密码
/usr/local/mysql/bin/mysql -e "grant all privileges on *.* to root@'127.0.0.1' identified by \"$dbrootpwd\" with grant option;"
/usr/local/mysql/bin/mysql -e "grant all privileges on *.* to root@'localhost' identified by \"$dbrootpwd\" with grant option;"
```
