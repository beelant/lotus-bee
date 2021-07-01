# lotus-Bee
**一个更适合矿工的Lotus**

[![standard-readme compliant](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)   ![version Bee](https://img.shields.io/badge/version-1.10.0-orange)

## 关于Lotus-Bee
[Lotus代码](https://github.com/filecoin-project/lotus/#project-lotus---%E8%8E%B2)在Protocol Labs近一年多的努力下，官方代码仍然拥有诸多缺陷，蜂灯科技凭借分布式存储领域开发经验和对[Lotus代码](https://github.com/filecoin-project/lotus/#project-lotus---%E8%8E%B2)的深刻钻研，特此开发Lotus-Bee版本Lotus代码，提供广大分布式存储提供商更快捷的数据封装速度，更灵活的集群配置方式，更稳定的运维特性的Lotus代码，为推动行业发展贡献绵薄之力，现决定开放免费使用。

## 主要增强功能
**已完成（已通过测试稳定运行）：**
1.    分离miner的三种角色，sealing、WinPost、WdPost，每个模块可以独立运行，互不干扰。大算力情况下增加爆块率。（⚠️不要开启两个及以上的WinPost机器，容易出现分叉罚款。WdPost可以多开，保证百分百不掉算力）

2.    设定worker各个任务的最大并发数，用于适配不同机器的可用内存，可用于异机架构，多种机器混合部署。（根据不同机器内存大小配置不同的任务比例，⚠️建议分配任务时不要考虑swap缓存，会极大的影响作业效率，默认所有可接受任务数量是0）

3.    设定worker工作区的MaxSectors并发作业的总数量，用于不同机型的nvme缓存盘大小，防止爆盘。

4.    可命令开启和关闭的全自动智能add piece策略，保证机器不间断运行，也不会扇区超量，也可以随时停止，也可根据余额来。

5.    可以任务单机调度都在同一台worker机器上执行，避免所有的文件传输，智能调度保证机器满负载运行。可横向扩容worker，没有任何瓶颈，也可以让C2外包。

6.    worker自动同步miner存储路径，worker计算好扇区后直接落盘到存储，不需要经过miner回传。

7.    调整任务状态机，将落盘的过程提前到c2消息上链之前，防止上链之后落盘未完做时空证明出现错误扇区。

8.    数据落盘适配常规NFS，七牛云对象存储等。增加数据落盘校验，数据落盘后校验文件，确保文件无误。

9.    手动设定消息费cap值，根据实际情况进行调整多种关键消息的cap值。

10.    PC2,C2算法优化.

11.    部分数据落db表.（核心逐步迁移到db中）.

12.    批量提交precommit和commit消息逻辑优化，相关配置转移到数据库中（实时读取）,新增参数进入队列等待多久后batch时间。

**测试中：**

待更新

## 部署流程
1.    daemon（配置方案同官方，无特殊调整）

    lotus daemon

2.    miner

> * Miner 启动参数
   
>>   * --wdpost=true   是否开启WindowPost

>>  * --wnpost=true    是否开启WinningPost

>>   * --workername=master

>>   * --allow-c2-py=true    是否开启C2 外包

>> * --enable-db=true	是否开启db调度（不开启不会封装扇区）

2.1 Sealing Miner
* 初始化Sealing Miner
```shell
lotus-miner init --owner=<钱包地址> --sector-size=*GiB
sed -i 's/\"CanStore\": true/\"CanStore\": false/' $LOTUS_MINER_PATH/sectorstore.json
```
* 修改"./lotusminer/config.toml"文件
    ListenAddress = "/ip4/0.0.0.0/tcp/2345/http"
    RemoteListenAddress = "<本机ip>:2345"
* 配置Mysql（<font color=red>Sealing Miner 必须开启Mysql存储数据，其他Miner可不链接Mysql.</font>）
    I. 添加环境变量 SQL_PATH=YOUR_PATH/sql.json

    II. 新建mysql配置文件 sql.json
    ```json
    {
   "name": "name",
   "addr": "addr",
   "db": "db",
   "username": "username",
   "password": "password",
   "maxIdleConn": 0,
   "maxOpenConn": 100,
   "connMaxLifetime": 0
    }
    ```

*  运行Sealing Miner
```shell
nohup lotus-miner run --wdpost=false --wnpost=false --workername=master --enable-db=true	--allow-c2-py=true > miner.log 2>&1 &

```
* 挂载（使用存储分离 nsf挂载）
```shell
lotus-miner storage attach --init --store /home/nfs

```
2.2 WnPost Miner
* 拷贝Sealing Miner机器的Storage
* 推荐使用命令 （最优方案是miner数据放到分布式存储上面，确保数据不丢失，miner机器远程挂载miner数据）
```shell
rsync -av --exclude  storage/kvlog  --exclude storage/journal storage storage-wn
```
* 修改 ".lotusminer/config.toml"
RemoteListenAddress = "<本机ip>:端口"
* 运行WnPost Miner
```shell
nohup lotus-miner run --wdpost=false --wnpost=true --p2p=false --enable-db=false > lotus-wn.log 2>&1 &
```
2.3 WdPost Miner
* 拷贝Sealing Miner机器的".lotusminer"
```shell
rsync -av --exclude  storage/kvlog  --exclude storage/journal storage storage-wd

```
* 修改".lotusminer/config.toml"
RemoteListenAddress = "<本机ip>:端口"
* 运行WdPost Miner
```shell
nohup lotus-miner run --wdpost=true --wnpost=false --p2p=false --enable-db=false > lotus-wd.log 2>&1 &
```
3.    Worker
* 运行Worker（每种任务数量根据机器情况进行相应调整）
```shell
nohup lotus-worker  --listen=<本机ip>:3456 --ability=AP:1,MaxSector:12,PC1:6,PC2:1,C1:1,C2:1,FIN:1,GET:1,UNS:1,RD:1 > worker.log 2>&1 &
```
* ⚠️注意：worker的存储需挂载与miner一样的存储，相同的路径和相同的机器

4.    在sealing miner机器上开启自动质押
```shell
lotus-miner sectors auto on/off
lotus-miner sectors balance 100 //可以自由调整，worker余额低于这个数值的时候不会开启自动质押
```

## 运维特性/注意事项
1.    关于存储扩容问题
由于lotus集群对于外部存储的消耗较大，需要逐步增加挂载到集群的存储空间，如果需要增加不同的Path路径，需要首先在sealing miner上初始化：
```shell
lotus-miner storage attach --init=true /YOUR_STORAGE_PATH
```
然后在WnPost和WdPost上执行命令：
```shell
lotus-miner storage attach --init=false /YOUR_STORAGE_PATH

```
随后在集群中的所有worker上执行命令：
```shell
lotus-worker syncStore
```
（注意，需要等待sealing miner和worker正常进入服务状态之后）
2.    关于Miner的角色划分问题
miner需要承担sealing、WnPost、WdPost三种角色

目前的最佳实践是三种角色全部拆分，这样是最稳定的。

如果集群规模较小（worker数量小于100台，已有算力小于3P）可以将sealing和WnPost合并在一起。  
***  
任务并发数设定
MaxSector：取决于worker工作路径的空间大小，每一个32G的Sector需要消耗650GB的存储空间  
P1：取决于实际内存大小  
P2、C2: RUST优化还需要一些时间测试，目前阶段写1  
C1、AP、FIN、GET、UNS、RD写1即可  
| Worker机器规格 | 32G扇区推荐参数 | 64G推荐参数 |
| :-----| :----- | :---- |
| CPU：3960X,内存：256G,GPU：2080Ti,nvme：4T | AP:1,MaxSector:6，PC1:3,PC2:1,C1:1,C2:1,FIN:1，GET:1,UNS:1,RD:1 | 效率较低，不推荐 |
| CPU：7542,内存：1T,GPU：3080,nvme：16T | AP:1,MaxSector:24，PC1:12,PC2:1,C1:1,C2:1,FIN:1，GET:1,UNS:1,RD:1 | AP:1,MaxSector:12，PC1:6,PC2:1,C1:1,C2:1,FIN:1，GET:1,UNS:1,RD:1 |
3.    GasCap设定  
```shell
lotus setGasCap --preGasCap=1 --preBGasCap=1 --proGasCap=1 --proAGasCap=1 --subGasCap=10

```
上述命令含义设置
> * preGasCap     PreCommitSector 消息费设置默认1  NanoFIl
> * preBGasCap    PreCommitSectorBatch 消息费设置默认1  NanoFIl
> * proGasCap     ProveCommitSector 消息费设置默认1  NanoFIl
> * proAGasCap    ProveCommitAggregate 消息费设置默认1  NanoFIl
> * subGasCap     SubmitWindowedPoSt 消息费设置默认10 NanoFIl

4.    配置批量提交消息
```shell
lotus-miner config modify --BatchPreCommits=true --MaxPreCommitBatch=256
```  
也可以通过修改db配置 config
| 参数 | 含义 | 默认值 |
| :-----| :----- | :----- |
| BatchPreCommits | 是否开启批量提交precommit | True |
| MaxPreCommitBatch | 最大批量提交precommit数量 | 256 |
| MinPreCommitBatch | 最小批量提交precommit数量（暂未使用） | 1 |
| PreCommitBatchWait | 进入队列等待多久后batchPreCommit时间（单位：分钟） | 10 |
| PreCommitBatchSlack | 最晚结束前多少时间内提交precommit（单位：分）(暂未使用） | 360 |
| AggregateCommits | 是否开启批量提交commit | True |
| MinCommitBatch | 最小批量提交commit数量 | 4 |
| MaxCommitBatch | 最大批量提交commit数量 | 819 |
| CommitBatchWait | 进入队列等待多久后batchCommit时间（单位：分钟） | 10 |
| CommitBatchSlack | 最晚结束前多少时间内提交commit（单位：分 )(暂未使用） | 180 |





