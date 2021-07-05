# lotus-Bee
**A possibly better Lotus**

[![standard-readme compliant](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)   ![version Bee](https://img.shields.io/badge/version-1.10.2-orange)

## About Lotus-Bee
Beelant with distributed storage field development experience and familiarity with [Lotus](https://github.com/filecoin-project/lotus/#project-lotus---%E8%8E%B2), the development of Lotus-Bee version of Lotus code, to provide the majority of distributed storage providers faster data encapsulation speed, more flexible cluster configuration method, more stable operation and maintenance characteristics of Lotus code, to promote the development of the industry to make a small contribution.

## Optimised functions
**Available (tested and stable operation) ：**
1.    Splitting miner into three, Sealing, WinPost, WdPost, each module can run independently without interfering with each other. Increase block generation rate in case of large  Storage power. (⚠️ Do not turn on two and more WinPost machines, prone to fork fines. WdPost can be turned on more often to guarantee 100% no lost Storage Power)

2.    Set the maximum number of concurrent tasks for each worker, used to adapt the available memory of different machines, can be used for heterogeneous architecture, mixed deployment of multiple machines. (Configure different task ratios according to the memory size of different machines, ⚠️ it is recommended to not consider swap cache when assigning tasks, it will greatly affect the efficiency of the job, the default number of all acceptable tasks is 0)

3.    Set the total number of concurrent MaxSectors jobs in the worker workspace for different models of nvme cache disk sizes to prevent disk blowouts.

4.    A fully automatic intelligent add piece policy that can be commanded on and off ensures that the machine runs uninterrupted and also does not sector overload, and can also be stopped at any time, also depending on the balance.

5.    The option of single machine scheduling of tasks all executing on the same worker machine, avoiding all file transfers and intelligent scheduling to ensure the machine runs at full load. Horizontal scaling of workers is possible without any bottlenecks, also allowing C2 to be outsourced.

6.    The worker automatically synchronises the miner storage path, and the worker wraps the sectors and transfers them directly to the storage system without going back through the miner.

7.    Adjust the task state machine to advance the process of sealing the sector to before the C2 message is on the chain to prevent incorrect sectors after the chain is on and the drop disk is not finished doing the POST.

8.    The data storage is adapted to conventional NFS, Qiniu cloud object storage, etc. Add data drop verification, verify files after data drop to ensure they are error free.

9.    Manually set the message fee cap value and adjust the cap value for a wide range of key messages according to the actual situation.

10.    PC2,C2 algorithm optimisation.

11.    Partial data drop mysql tables. (The core is gradually migrated to mysql).

12.    Batch commit precommit and commit message logic optimization, related configuration transfer to the database (real-time reading), new parameters into the queue waiting for how long after batch time.

**In Test：**
1. Hardware Wallet Management Node.

## Configuration

DEV：Ubuntu18.04,AMD CPU
1.    daemon（The configuration scheme is the same as the official code, with no special adjustments.）

    lotus daemon

2.    miner

> * Miner Startup parameters
   
>>   * --wdpost=true   //Whether to enable WindowPost

>>  * --wnpost=true    //Whether to enable WinningPost

>>   * --workername=master

>>   * --allow-c2-py=true    //Whether to turn on C2 outsourcing

>> * --enable-db=true	  //Whether to turn on db scheduling (it will not seal without it turning on)


2.1 Sealing Miner
* Initialising Sealing Miner
```shell
lotus-miner init --owner=<Wallet address> --sector-size=*GiB
sed -i 's/\"CanStore\": true/\"CanStore\": false/' $LOTUS_MINER_PATH/sectorstore.json
```
* Modify `./lotusminer/config.toml`
```shell
    ListenAddress = "/ip4/0.0.0.0/tcp/2345/http"

    RemoteListenAddress = "<local ip>:2345"
```    
* Configure Mysql（Sealing Miner must have Mysql data storage enabled, other Miners may not be linked to Mysql.）

    I. Adding environment variables  `SQL_PATH=YOUR_PATH/sql.json`

    II. Create a new mysql configuration file `sql.json`
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

*  Running Sealing Miner
```shell
nohup lotus-miner run --wdpost=false --wnpost=false --workername=master --enable-db=true	--allow-c2-py=true > miner.log 2>&1 &

```
* Mounting（Use storage separation  nsf mount）
```shell
lotus-miner storage attach --init --store /home/nfs

```
2.2 WnPost Miner
* Copy Storage from Sealing Miner 

* Recommended commands (optimal solution is to put miner data on top of distributed storage to ensure no data loss, miner machine remotely mounts miner data)

```shell
rsync -av --exclude  storage/kvlog  --exclude storage/journal storage storage-wn
```
* Modify `.lotusminer/config.toml`
```shell
RemoteListenAddress = "<Local ip>:Port"


```
* Running WnPost Miner
```shell
nohup lotus-miner run --wdpost=false --wnpost=true --p2p=false --enable-db=false > lotus-wn.log 2>&1 &
```
2.3 WdPost Miner
* Copy `.lotusminer` from Sealing Miner 
```shell
rsync -av --exclude  storage/kvlog  --exclude storage/journal storage storage-wd

```
* Modify`.lotusminer/config.toml`
```shell
RemoteListenAddress = "<Local ip>:Port"
```

* Running WdPost Miner
```shell
nohup lotus-miner run --wdpost=true --wnpost=false --p2p=false --enable-db=false > lotus-wd.log 2>&1 &
```
3.    Worker
* Running Worker（The number of each type of task is adjusted accordingly to the machine）
```shell
nohup lotus-worker  --listen=<local ip>:3456 --ability=AP:1,MaxSector:12,PC1:6,PC2:1,C1:1,C2:1,FIN:1,GET:1,UNS:1,RD:1 > worker.log 2>&1 &
```
* ⚠️Note: The worker storage needs to be mounted on the same storage as the miner, with the same path and the same machine.

4.    Enable automatic pledging on sealing miner 
```shell
lotus-miner sectors auto on/off
lotus-miner sectors balance 100 //Freely adjustable, no automatic pledging will be activated if the worker balance falls below this value
```

## O&M features / notes
1.    About storage expansion

As lotus clusters consume a lot of external storage, it is necessary to gradually increase the storage space mounted to the cluster, and if different Path paths need to be added, they need to be initialized on the sealing miner first：
```shell
lotus-miner storage attach --init=true /YOUR_STORAGE_PATH
```
Then execute the command on WnPost and WdPost：

```shell
lotus-miner storage attach --init=false /YOUR_STORAGE_PATH

```
Then execute the command on all workers in the cluster：
```shell
lotus-worker syncStore
```
（⚠️Note: You need to wait for the sealing miner and worker to enter the service state normally.）

2.    About Miner's role division

miner needs to take on the roles of Sealing, WnPost and WdPost

The current best practice is to split all three roles, which is the most stable.

If the cluster size is small (less than 100 workers, less than 3Pb of existing storage power ), you can combine sealing and WnPost together.   
***  
Task concurrency setting
MaxSector：Depending on the space size of the worker's work path, each 32G Sector consumes 650GB of storage space.
P1：Depends on how much memory is actually available  
P2、C2: RUST optimisation still needs some time to test, at this stage optimising P1.  
C1、AP、FIN、GET、UNS、RD just write 1  
| Worker | 32G Parameter | 64G Parameter |
| :-----| :----- | :---- |
| CPU：3960X,RAM：256G,GPU：2080Ti,nvme：4T | AP:1,MaxSector:6，PC1:3,PC2:1,C1:1,C2:1,FIN:1，GET:1,UNS:1,RD:1 | Less efficient, not recommended |
| CPU：7542,RAM：1T,GPU：3080,nvme：16T | AP:1,MaxSector:24，PC1:12,PC2:1,C1:1,C2:1,FIN:1，GET:1,UNS:1,RD:1 | AP:1,MaxSector:12，PC1:6,PC2:1,C1:1,C2:1,FIN:1，GET:1,UNS:1,RD:1 |
3.    GasCap setting  
```shell
lotus setGasCap --preGasCap=1 --preBGasCap=1 --proGasCap=1 --proAGasCap=1 --subGasCap=10

```
Command Meaning
> * preGasCap=1:     PreCommitSector gasfee default 1  NanoFil
> * preBGasCap=1:    PreCommitSectorBatch gasfee default 1  NanoFil
> * proGasCap=1:     ProveCommitSector gasfee default 1  NanoFil
> * proAGasCap=1:    ProveCommitAggregate gasfee default 1  NanoFil
> * subGasCap=10:    SubmitWindowedPoSt gasfee default 10 NanoFil

4.    Configuring bulk submission messages
```shell
lotus-miner config modify --BatchPreCommits=true --MaxPreCommitBatch=256
```  
can also be done by modifying the db configuration config
| Parameters | mean | Default value |
| :-----| :----- | :----- |
| BatchPreCommits | Whether to enable batch commit precommit  | True |
| MaxPreCommitBatch | Maximum number of batch commits precommit | 256 |
| MinPreCommitBatch | Minimum number of batch commit precommits (not used yet) | 1 |
| PreCommitBatchWait | How long to wait after entering the queue batchPreCommit time (in minutes) | 10 |
| PreCommitBatchSlack | How many minutes before the end of the latest time to submit a precommit (in minutes) (not used yet) | 360 |
| AggregateCommits | Whether to enable batch commit | True |
| MinCommitBatch | Minimum number of batch commits | 4 |
| MaxCommitBatch | Maximum number of batch commits | 819 |
| CommitBatchWait | How long to wait in the queue before batchCommit time (in minutes) | 10 |
| CommitBatchSlack | How many minutes before the end of the latest time to submit a commit (in minutes ) (not used yet) | 180 |



## Discuss in WeChat
![image](https://user-images.githubusercontent.com/86239661/124066300-04187680-da6b-11eb-91b7-9cd92bf60540.png)




