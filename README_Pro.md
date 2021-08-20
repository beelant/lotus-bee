# Lotus-BeePro

**一个比Lotus-Bee更快更安全的Lotus**

## 从Lotus-Bee进阶优化

1. 配置优化：

    - 几乎所有数据落表DB，集群所有可选择配置实时生效，灵活配置。   

2. 存储优化：  
    - 针对对象存储定制优化  
    - 支持大规模节点   

3. 算法优化：

    - PreCommit2和Commit2任务完成时间消耗10min以内。
    - 支持多卡随意搭配允许多个gpu任务。例：双卡机可同时进行2个c2或者 2个p2 或者 1个p2 1个c2等等以此类推。   
    
    - 以下为**32Gb扇区 7542 3080*1** benchmark数据
```
results (v28) SectorSize:(34359738368), SectorNumber:(1)
seal: addPiece: 1m19.866417797s (410.3 MiB/s)
seal: preCommit phase 1: 2h48m36.947479519s (3.239 MiB/s)
seal: preCommit phase 2: 8m29.863756006s (64.27 MiB/s)
seal: commit phase 1: 428.302455ms (74.71 GiB/s)
seal: commit phase 2: 11m8.230462756s (49.04 MiB/s)
seal: verify: 12.726156ms

generate candidates: 178.612µs (175 TiB/s)
compute winning post proof (cold): 4.148707004s
compute winning post proof (hot): 3.750776236s
verify winning post proof (cold): 86.592725ms
verify winning post proof (hot): 6.403544ms

compute window post proof (cold): 2m29.364263484s
compute window post proof (hot): 1m49.256524931s
verify window post proof (cold): 8.38895788s
verify window post proof (hot): 17.095116ms
```
4. 调度深度优化：

    - 集群调度优化后，更快更稳定，任务塞的更满。
    - 适配各种机型。   

**效率大幅度提升，实际测试对比相同配置的机器比免费版提升约60%-100%**

5. 远程硬件钱包支持，支持在家调用硬件钱包远程管理机房节点 **保护老板数据资产安全** 。

   （最近安全事故频发，此功能完美避免内外安全风险，无论是外部入侵还是内部隐患都无法窃取节点，让节点老板能睡个安稳觉）  
   
6. 多维度优化扇区状态机，避免扇区过期、失效、重复执行某个步骤等各种问题，以防卡住机器卡住调度的情况出现

7. 智能预警系统，worker机器出现硬件问题系统无法自动解决时，及时提醒报警

8. lotus集群化管理部署，多lotus智能负载均衡，确保大集群不会出现单点故障

## 等待合并功能  

1. 多wd机器分担任务调度。  

## 从lotus-bee 继承过来的功能
1.    分离miner的三种角色，sealing、WinPost、WdPost，每个模块可以独立运行，互不干扰。大算力情况下增加爆块率。（⚠️不要开启两个及以上的WinPost机器，容易出现分叉罚款。WdPost可以多开，保证百分百不掉算力）

2.    设定worker各个任务的最大并发数，用于适配不同机器的可用内存，可用于异机架构，多种机器混合部署。（根据不同机器内存大小配置不同的任务比例，⚠️建议分配任务时不要考虑swap缓存，会极大的影响作业效率，默认所有可接受任务数量是0）

3.    设定worker工作区的MaxSectors并发作业的总数量，用于不同机型的nvme缓存盘大小，防止爆盘。

4.    可命令开启和关闭的全自动智能add piece策略，保证机器不间断运行，也不会扇区超量，也可以随时停止，也可根据余额来。

5.    可以任务单机调度都在同一台worker机器上执行，避免所有的文件传输，智能调度保证机器满负载运行。可横向扩容worker，没有任何瓶颈，也可以让C2外包。

6.    worker自动同步miner存储路径，worker计算好扇区后直接落盘到存储，不需要经过miner回传。

7.    调整任务状态机，将落盘的过程提前到c2消息上链之前，防止上链之后落盘未完做时空证明出现错误扇区。（与官方配置无关）

8.    数据落盘适配常规NFS，七牛云对象存储等。增加数据落盘校验，数据落盘后校验文件，确保文件无误。

9.    手动设定消息费cap值，根据实际情况进行调整多种关键消息的cap值。

10.    PC2多张显卡可以同时分别计算多个PC2，C2算法优化。

11.    部分数据落mysql表.（核心逐步迁移到mysql中）。

12.    批量提交precommit和commit消息逻辑优化，相关配置转移到数据库中（实时读取）,新增参数进入队列等待多久后batch时间。

## 商务洽谈
![image](https://user-images.githubusercontent.com/86239661/126980665-1ca8c996-c9c1-4c86-b585-0ed42feebf77.png)



