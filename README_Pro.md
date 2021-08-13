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

## 等待合并功能

1. lotus集群化管理。  

2. 多wd机器分担任务调度。  



## 商务洽谈
![image](https://user-images.githubusercontent.com/86239661/126980665-1ca8c996-c9c1-4c86-b585-0ed42feebf77.png)



