Clone source code from this repo.

编译安装

$ ./configure --prefix=/appdb/pgfde --with-readline

$ make world -j 32

$ make install-world
设置加密密钥，并初始化数据库集群

$ read -sp "Postgres passphrase: " PGENCRYPTIONKEY
$ export PGENCRYPTIONKEY
$ echo $PGENCRYPTIONKEY
$ initdb --data-encryption pgcrypto --data-checksums -D /data/fde -E UTF8 --locale=C -U xdb
启动数据库

$ /appdb/pgfde/bin/pg_ctl -D /data/fde -l /data/fde/pg-fde.log start
必须设置好PGENCRYPTIONKEY这个环境变量,否则数据库启动不起来

$ tail -f /data/fde/pg-fde.log

...
LOG:  encryption key not provided
DETAIL:  The database cluster was initialized with encryption but the server was started without an encryption key.
HINT:  Set the key using PGENCRYPTIONKEY environment variable.
FATAL:  data encryption could not be initialized
LOG:  database system is shut down
正常启动

$ /appdb/pgfde/bin/pg_ctl -D /data/fde -l /data/fde/serverlog status
pg_ctl: server is running (PID: 1471)
/appdb/pgfde/bin/postgres "-D" "/data/fde"
测试

$ psql -d testdb -p 5433
psql (9.6.9, server 9.6.0)
Type "help" for help.

testdb=# create table test(id int, c1 char(8),c2 varchar(16));
CREATE TABLE
testdb=# select pg_relation_filepath('test');
 pg_relation_filepath 
----------------------
 base/16384/16388
(1 row)

testdb=# --插入2条数据
testdb=# insert into test values (1,'1','1');
INSERT 0 1
testdb=# insert into test values (2,'2','2');
INSERT 0 1
testdb=# insert into test values (3,'c','c');
INSERT 0 1
testdb=# --执行checkpoint，使Shared Buufer的数据刷入到磁盘中
testdb=# checkpoint ;
CHECKPOINT
