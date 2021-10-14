# lotus-Bee
[ENGLISH](https://github.com/beelant/lotus-bee/blob/main/README_en.md)

**一个可能更好用的Lotus**

[![standard-readme compliant](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)   ![version Bee](https://img.shields.io/badge/version-1.10.3-orange)

## 关于Lotus-Bee
Beelant Tech凭借分布式存储领域开发经验和对[Lotus代码](https://github.com/filecoin-project/lotus/#project-lotus---%E8%8E%B2)的深刻钻研，特此开发Lotus-Bee版本Lotus代码，提供广大分布式存储提供商更快捷的数据封装速度，更灵活的集群配置方式，更稳定的运维特性的Lotus代码，为推动行业发展贡献绵薄之力，现决定开放免费使用。

## 部署文档
[部署文档](https://github.com/beelant/lotus-bee/blob/main/deployment.md)

## 主要功能


1. 运维优化：
    - 从官方升级无忧，回退官方也无忧
    - 分布式miner、高效率worker
    - 所有核心数据落表DB，实时监控
    - 集群所有可选择配置实时生效，灵活配置。   
    - 超多实用配置，手续费、抵押币等各种灵活操作

2. 存储优化：  
    - 针对对象存储定制优化，高效读写
    - 支持大规模节点，单节点轻松到达100P   

3. 算法优化：
    - PreCommit2和Commit2任务完成时间消耗10min以内。
    - 支持多卡随意搭配允许多个gpu任务。例：双卡机可同时进行2个c2或者 2个p2 或者 1个p2 1个c2等等以此类推。   

4. 调度深度优化：
    - 集群调度大幅度优化，更快更稳定，解锁满载状态
    - 适配各种机型，无论是3系，7系，256GB内存、1T内存、2T内存完美兼容
    - 32GB扇区 64GB扇区完美兼容

5. 安全优化
   - 远程硬件钱包支持，支持在家调用硬件钱包远程管理机房节点 
   - 钱包服务独立，可独立于集群做好安全措施
   
6. 扇区优化
   - 多维度优化扇区状态机，避免扇区过期、失效、重复执行某个步骤等各种问题
   - 随时验证扇区完整，排查问题
   
7. daemon集群化
   - 集群化管理部署，多daemon智能负载均衡，确保大集群不会出现单点故障
   - 灵活剪枝，剪枝过程中miner正常运行


8. windowPost集群化
   - 多wd机器，智能负载均衡分担时空证明调度
   - 突破单台时空证明算力上限，理论无上限
   - 避免单点故障，任意机器故障不影响时空证明




## 交流分享
![image](https://user-images.githubusercontent.com/86239661/131319877-42444f39-fde2-4b2e-a48b-6f1f1693c105.png)


如显示已满请添加商务洽谈微信，回复：拉群

欢迎加入我们的技术讨论Discord群：     
https://discord.gg/Vb7eFHkPzB





## 商务洽谈
![image](https://user-images.githubusercontent.com/86239661/126980665-1ca8c996-c9c1-4c86-b585-0ed42feebf77.png)











