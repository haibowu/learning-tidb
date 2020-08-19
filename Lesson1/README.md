##编译tidb
直接在项目根目录使用make命令就可以编译成功，本地的LDFLAGS环境变量有别的值，导致编译失败，所以修改了下Makefile文件，
将LDFLAGS先置空

##启动tidb
为了方便本机直接登录，所以在config文件里面加了skip-grant-table=true，启动命令如下：
sudo ./tidb-server --log-file="tidbRunning.log" --path="127.0.0.1:2379" --store="tikv" --config="/Users/haibo/tidb/config/config.toml"


##编译pd
直接在项目根目录直接make编译，启动命令如下：
./bin/pd-server --name=pd1 --data-dir=pd1 --client-urls="http://127.0.0.1:2379" --peer-urls="http://127.0.0.1:2380" --initial-cluster="pd1=http://127.0.0.1:2380" --log-file=pd1.log


##编译tikv
直接在项目根据路使用make build命令编译，因为config文件的max-open-files设置的很大，启动会报错
[FATAL] [server.rs:920] ["the maximum number of open file descriptors is too small, got 2560, expect greater or equal to 82920"]
所以修改了配置文件config.rs中的配置，编译好后启动3个tikv实例：
./tikv-server --pd-endpoints="127.0.0.1:2379" --addr="127.0.0.1:30160" --data-dir=tikv1 --log-file=tikv1.log &
./tikv-server --pd-endpoints="127.0.0.1:2379" --addr="127.0.0.1:30161" --data-dir=tikv2 --log-file=tikv2.log &
./tikv-server --pd-endpoints="127.0.0.1:2379" --addr="127.0.0.1:30162" --data-dir=tikv3 --log-file=tikv3.log &

使用
./bin/pd-ctl store -u http://127.0.0.1:2379
获取tikv状态数据
{
  "count": 3,
  "stores": [
    {
      "store": {
        "id": 1,
        "address": "127.0.0.1:30162",
        "version": "4.1.0-alpha",
        "status_address": "127.0.0.1:20180",
        "git_hash": "ae7a6ecee6e3367da016df0293a9ffe9cc2b5705",
        "start_timestamp": 1597754722,
        "deploy_path": "/Users/haibo/tikv/target/debug",
        "last_heartbeat": 1597807288799863000,
        "state_name": "Up"
      },
      "status": {
        "capacity": "185.3GiB",
        "available": "36.58GiB",
        "used_size": "30.04MiB",
        "leader_count": 0,
        "leader_weight": 1,
        "leader_score": 0,
        "leader_size": 0,
        "region_count": 0,
        "region_weight": 1,
        "region_score": 1073704370.3632812,
        "region_size": 0,
        "start_ts": "2020-08-18T20:45:22+08:00",
        "last_heartbeat_ts": "2020-08-19T11:21:28.799863+08:00",
        "uptime": "14h36m6.799863s"
      }
    },
    {
      "store": {
        "id": 4,
        "address": "127.0.0.1:30161",
        "version": "4.1.0-alpha",
        "status_address": "127.0.0.1:20180",
        "git_hash": "ae7a6ecee6e3367da016df0293a9ffe9cc2b5705",
        "start_timestamp": 1597754655,
        "deploy_path": "/Users/haibo/tikv/target/debug",
        "last_heartbeat": 1597807288780379000,
        "state_name": "Up"
      },
      "status": {
        "capacity": "185.3GiB",
        "available": "36.58GiB",
        "used_size": "28.8MiB",
        "leader_count": 0,
        "leader_weight": 1,
        "leader_score": 0,
        "leader_size": 0,
        "region_count": 0,
        "region_weight": 1,
        "region_score": 1073704370.3632812,
        "region_size": 0,
        "start_ts": "2020-08-18T20:44:15+08:00",
        "last_heartbeat_ts": "2020-08-19T11:21:28.780379+08:00",
        "uptime": "14h37m13.780379s"
      }
    },
    {
      "store": {
        "id": 5,
        "address": "127.0.0.1:30160",
        "version": "4.1.0-alpha",
        "status_address": "127.0.0.1:20180",
        "git_hash": "ae7a6ecee6e3367da016df0293a9ffe9cc2b5705",
        "start_timestamp": 1597754417,
        "deploy_path": "/Users/haibo/tikv/target/debug",
        "last_heartbeat": 1597807288891331000,
        "state_name": "Up"
      },
      "status": {
        "capacity": "185.3GiB",
        "available": "36.58GiB",
        "used_size": "28.8MiB",
        "leader_count": 0,
        "leader_weight": 1,
        "leader_score": 0,
        "leader_size": 0,
        "region_count": 0,
        "region_weight": 1,
        "region_score": 1073704370.3632812,
        "region_size": 0,
        "start_ts": "2020-08-18T20:40:17+08:00",
        "last_heartbeat_ts": "2020-08-19T11:21:28.891331+08:00",
        "uptime": "14h41m11.891331s"
      }
    }
  ]
}

使用mysql -h 127.0.0.1 -P 4000可以正常连接数据库
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.7.25-TiDB-v4.0.0-beta.2-524-g73fe3ca4d-dirty TiDB Server (Apache License 2.0) Community Edition, MySQL 5.7 compatible

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
mysql> 
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| INFORMATION_SCHEMA |
| METRICS_SCHEMA     |
| PERFORMANCE_SCHEMA |
| mysql              |
| test               |
+--------------------+
5 rows in set (0.01 sec)



##启动tidb事务的时候可以打印"hello transaction"
根据https://pingcap.com/blog-cn/tidb-source-code-reading-18/这一章关于tidb中的解析，Storage的实现tikvStore中，
Begin() (Transaction, error)和BeginWithStartTS(startTS uint64) (Transaction, error)方法都会调用
func newTikvTxnWithStartTS(store *tikvStore, startTS uint64, replicaReadSeed uint32) (*tikvTxn, error)
因此在该方法里加上日志输出
logutil.BgLogger().Info("hello transaction")




