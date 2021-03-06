# Aggregator

集群聚合模块。聚合某集群下的所有机器的某个指标的值，提供一种集群视角的监控体验。

## 准备工作

该模块是后来加入的，检查你的portal中是否有这个代码：https://github.com/open-falcon/portal/blob/master/web/model/cluster.py，如果有了，说明版本OK，否则，需要升级原来的portal为最新版代码。

falcon_portal数据库中加入了一张新表：

```sql
USE falcon_portal;
SET NAMES 'utf8';
 
DROP TABLE IF EXISTS cluster;
CREATE TABLE cluster
(
  id          INT UNSIGNED   NOT NULL AUTO_INCREMENT,
  grp_id      INT            NOT NULL,
  numerator   VARCHAR(10240) NOT NULL,
  denominator VARCHAR(10240) NOT NULL,
  endpoint    VARCHAR(255)   NOT NULL,
  metric      VARCHAR(255)   NOT NULL,
  tags        VARCHAR(255)   NOT NULL,
  ds_type     VARCHAR(255)   NOT NULL,
  step        INT            NOT NULL,
  last_update TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  creator     VARCHAR(255)   NOT NULL,
  PRIMARY KEY (id)
)
  ENGINE =InnoDB
  DEFAULT CHARSET =latin1;

```

## 源码编译

```bash
cd $GOPATH/src/github.com/open-falcom
git clone https://github.com/open-falcon/sdk.git
git clone https://github.com/open-falcon/aggregator.git
cd aggregator
go get ./...
./control build
./control pack
```

最后一步会pack出一个tar.gz的安装包，拿着这个包去部署服务即可。

## 服务部署
服务部署，包括配置修改、启动服务、检验服务、停止服务等。这之前，需要将安装包解压到服务的部署目录下。

```bash
# 修改配置, 配置项含义见下文
mv cfg.example.json cfg.json
vim cfg.json

# 启动服务
./control start

# 校验服务，看端口是否在监听
ss -tln

# 检查log
./control tail

...
# 停止服务
./control stop

```


## 配置说明
配置文件默认为./cfg.json。默认情况下，安装包会有一个cfg.example.json的配置文件示例。各配置项的含义，如下

```bash
## Configuration
{
    "debug": true,
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:6055"
    },
    "database": {
        "addr": "root:@tcp(127.0.0.1:3306)/falcon_portal?loc=Local&parseTime=true",
        "idle": 10,
        "ids": [1,-1], # aggregator模块可以部署多个实例，这个配置表示当前实例要处理的数据库中cluster表的id范围
        "interval": 55
    },
    "api": {
        "hostnames": "http://127.0.0.1:5050/api/group/%s/hosts.json", # 注意修改为你的portal的ip:port
        "push": "http://127.0.0.1:6060/api/push", # 注意修改为你的transfer的ip:port
        "graphLast": "http://127.0.0.1:9966/graph/last" # 注意修改为你的query的ip:port
    }
}
       
```
