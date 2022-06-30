- [一、Lotus](#一lotus)
  - [基础环境](#基础环境)
  - [升级流程](#升级流程)
  - [部署说明](#部署说明)
  - [启动命令](#启动命令)
- [二、Miner](#二miner)
  - [准备环境](#准备环境)
  - [部署说明](#部署说明-1)
  - [Miner启动参数介绍](#miner启动参数介绍)
  - [sealing + WinningPost 模式](#sealing--winningpost-模式)
  - [仅WindowPost模式](#仅windowpost模式)
- [三、Worker](#三worker)
  - [环境变量](#环境变量)
  - [启动参数介绍，两种运行模式](#启动参数介绍两种运行模式)
    - [ability参数说明](#ability参数说明)
    - [单机完成全流程](#单机完成全流程)
    - [C2拆分模式](#c2拆分模式)
- [常用运维命令](#常用运维命令)
- [Daemon集群部署](#daemon集群部署)
  - [Daemon集群说明](#daemon集群说明)
  - [部署流程](#部署流程)
- [WindowPost集群部署](#windowpost集群部署)
  - [WD集群说明](#wd集群说明)
  - [WD集群部署](#wd集群部署)

# 一、Lotus

## 基础环境
*  OS： Ubuntu18.04
*  CPU：AMD 
*  内存： >= 256GB
*  与官方版本兼容性： 可来回切换
*  lotus-bee V1.16.0

## 升级流程
1. 下载lotus-bee 替换官方程序
2. 安装Mysql,创建新的库
3. 调整环境变量，增加所需要的配置
4. 根据官方文档初始化节点 （如已有节点请跳过）
5. 根据需要调整启动命令
6. 启动

## 部署说明
1. 环境参数 /etc/profile
```
export IPFS_GATEWAY=https://proof-parameters.s3.cn-south-1.jdcloud-oss.com/ipfs/
export FIL_PROOFS_MAXIMIZE_CACHING=1
export RUST_LOG=Info
export FIL_PROOFS_USE_GPU_COLUMN_BUILDER=1
export FIL_PROOFS_USE_GPU_TREE_BUILDER=1
export FIL_PROOFS_USE_MULTICORE_SDR=1
export GOLOG_LOG_FMT=json
export LOTUS_STORAGE_PATH=
export LOTUS_PATH=
export FIL_PROOFS_PARENT_CACHE=
```



## 启动命令
```
nohup lotus daemon > /home/daemon.log 2>&1 &
```





# 二、Miner
## 准备环境
* 如果有多台miner,需要每台miner上面都有miner的storage文件,从最开始的单miner机器上面拷贝即可
* 推荐拷贝命令，生成storage-wd文件夹之后 scp到新的miner-wd机器上面
```
rsync -av --exclude  storage/kvlog  --exclude storage/journal storage storage-wd
```
* 同一节点miner连接同一个数据库


## 部署说明
1. 环境参数 
```
export IPFS_GATEWAY=https://proof-parameters.s3.cn-south-1.jdcloud-oss.com/ipfs/
export LOTUS_STORAGE_PATH=/home/data/storage
export FIL_PROOFS_MAXIMIZE_CACHING=1
export FIL_PROOFS_USE_GPU_COLUMN_BUILDER=1
export FIL_PROOFS_USE_GPU_TREE_BUILDER=1
export FIL_PROOFS_PARENT_CACHE=/home/data/
export FIL_PROOFS_USE_MULTICORE_SDR=1
export LOTUS_PATH=/home/daemon
export FIL_PROOFS_SDR_PARENTS_CACHE_SIZE=1073741824
export GOLOG_LOG_FMT=json
export FIL_PROOFS_PARAMETER_CACHE=/var/tmp/filecoin-proof-parameters/
export RUST_LOG=Info
export GOLOG_LOG_LEVEL=Info
export SQL_PATH=/home/sql.json #数据库连接配置
```
2. 数据库连接配置模版(sql.json)文件
```
{
   "name": "beelant",
   "addr": "xxxx",
   "db": "beelant",
   "username": "xxxx", 
   "password": "xxxx",
   "maxIdleConn": 0,
   "maxOpenConn": 100,
   "connMaxLifetime": 0
}
```
3. 数据库创建语句
```
CREATE DATABASE `beelant` CHARACTER SET utf8 COLLATE utf8_general_ci;
```

## Miner启动参数介绍

* --wdpost=true    是否开启WindowPost 默认true
* --wnpost=true    是否开启WinningPost 默认true
* --workername=master    
* --enable-db=true	是否开启db调度（不开启不会封装扇区）默认true
* --is-schedule=true，是否开启调度，默认true，wdminer关闭
* --allow-c2-py=true，开启C2漂移，默认开启
* --p2p=false，接单模式，默认关闭
* --redeclare-sector=false 是否扫描存储中的扇区重新声明，存入mysql（默认不开，高阶功能，不建议随意使用）


## sealing + WinningPost 模式
```
nohup lotus-miner run --p2p=false --wdpost=false --wnpost=true --is-schedule=true --enable-db=true --workername=master > /home/miner.log 2>&1 &
```
## 仅WindowPost模式
```
nohup lotus-miner run --p2p=false --wdpost=true --wnpost=false --is-schedule=false --enable-db=true --workername=master > /home/miner.log 2>&1 &
```



# 三、Worker
## 环境变量
```
export IPFS_GATEWAY=https://proof-parameters.s3.cn-south-1.jdcloud-oss.com/ipfs/
export GOLOG_LOG_FMT=json
export RUST_LOG=Info
export GOLOG_LOG_LEVEL=Info
export FIL_PROOFS_MAXIMIZE_CACHING=1
export FIL_PROOFS_USE_GPU_COLUMN_BUILDER=1
export FIL_PROOFS_USE_GPU_TREE_BUILDER=1
export FIL_PROOFS_USE_MULTICORE_SDR=1
export FIL_PROOFS_SDR_PARENTS_CACHE_SIZE=1073741824
export FIL_PROOFS_MULTICORE_SDR_PRODUCERS=1
export WORKER_PATH=/home/data/worker
export FIL_PROOFS_PARENT_CACHE=/home/data/
export MINER_API_INFO=
```

## 启动参数介绍，两种运行模式
### ability参数说明
* MaxSector：根据worker工作路径的空间大小，每一个32G的Sector需要消耗650GB的存储空间
* P1：根据实际内存大小，32G的Sector占用64GB内存
* P2：根据显卡张数
* C2：根据显卡张数
* 其他：写1即可

### 单机完成全流程

```
nohup lotus-worker run --listen=本地IP:3456 --workername=1 --accept-other-c2=false --ability=AP:1,MaxSector:12,PC1:6,PC2:1,C1:1,C2:1,FIN:1,GET:1,UNS:1,RD:1 > /home/worker.log 2>&1 & 
```

### C2拆分模式

```
## 只做p1,p2,c1机器
nohup lotus-worker run --listen=本地IP:3456 --workername=1 --accept-other-c2=false  --ability=AP:1,MaxSector:12,PC1:6,PC2:1,C1:1,C2:0,FIN:1,GET:1,UNS:1,RD:1 > /home/worker.log 2>&1 & 
## 只做c2机器
nohup lotus-worker run --listen=本地IP:3456 --workername=1 --accept-other-c2=true  --ability=AP:0,MaxSector:0,PC1:0,PC2:0,C1:0,C2:4,FIN:0,GET:0,UNS:0,RD:0 > /home/worker.log 2>&1 & 
```

# 常用运维命令

* lotus-miner config list                   #查看miner配置
* lotus-miner config modify (configname)=?  #调整miner配置
* lotus-miner sealing workersRunInfo        #输出worker的任务信息
* lotus-miner sealing sched-diag            #输出调度的任务队列信息
* lotus-miner sealing abort sid=?           #删除对应work的track
* lotus-miner sectors postBee {sectorID}    #验证一个sector的wdpost
* lotus-miner sectors deadlinePost {deadlineIdx}    #验证一个deadline的wdpost 
* lotus-miner sectors remove ---really-do-it {sectorID} #强制删除sector
* lotus setGasCap --preGasCap=1 --preBGasCap=1 --proGasCap=1 --proAGasCap=1 --subGasCap=10  #设置gas cap，单位nanoFil

# Daemon集群部署

## Daemon集群说明

* 需要启动额外服务lotus-wallet，钱包都在lotus-wallet里面
* lotus-daemon机器不存任何钱包信息
* miner可同时连接多台daemon机器，自动负载均衡寻优

## 部署流程

 1. 数据库配置文件 ，daemon机和miner机上都需要配置，daemon-sql.json的模版
    
    ```
    {
    "name": "beelant",//
    "addr": "172.16.1.1", //数据库的访问地址
    "db": "f0123",//数据库名称
    "username": "beelant",//数据库账号
    "password": "beelant",//数据库密码
    "maxIdleConn": 0,
    "maxOpenConn": 100,
    "connMaxLifetime": 0
    }
    ```
    
 2. 环境变量配置，daemon机和miner机上都需要配置
    ```
    echo export DAEMON_CLUSTER=true >> /etc/profile         
    echo export DAEMON_SQL_PATH=/yourpath/daemon-sql.json  >> /etc/profile 
    source /etc/profile 
    ```

 3. daemon机器配置
 
    ```
    echo export DAEMON_URL=http://本地ip:1234 >> /etc/profile 
    source /etc/profile
    ```
    
 4. 启动lotus-wallet

    ```
    export WALLET_PATH=存储位置 >> /etc/profile #可选环境变量，不配置默认~/.lotuswallet
    nohup lotus-wallet run  --listen=本机ip:1777 > /home/wallet.log 2>&1 & 
    
    ```

 5. 修改lotus的config.toml，配置wallet连接信息
 
    ```
    [Wallet]
    RemoteBackend = "http://wallet-ip:1777"
    ```

 6. 同步daemon机器的 jwt_token,所有daemon机器需要保证同样的token
    * 在第一台启动好的daemon的工作目录下，会生成一个文件“beePrivateKey”
    * 将beePrivateKey和token复制到所有Daemon集群中的其他机器对应的位置上
    * 重启daemon服务
    * 所有miner使用第一台机器的token

 7. miner上配置环境变量，然后重启miner服务
    ```
    export FULLNODE_API_INFO=(lotus_jwt_token):/ip4/127.0.0.1/tcp/8888/http #这里ip一定是本机IP，端口是8888
    ```
    
# WindowPost集群部署

## WD集群说明
* 同一个集群的wd保证都能读取到存储文件
* 多台wd机器会在deadline自动接取不同的partition完成wd任务
* 系统自动分配，智能调控机器完成状态
* 单机优化状态下能完成5个partition以上


## WD集群部署
在多台已经配置完成wd的机器上增加启动参数--wd-colony
```
nohup lotus-miner run  --wdpost=true --wnpost=false --is-schedule=false --enable-db=true --wd-colony=true --workername=master > /home/miner.log 2>&1 &
```
