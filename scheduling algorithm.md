根据其调度时是否考虑后端主机的当前负载，可分为静态方法和动态方法两类

#### 静态方法：仅根据算法本身进行调度

- RR：Round Robin，轮询/轮调/轮叫；
- WRR：Weighted RR，加权轮询；根据服务器的性能配置权重进行调度
- SH：Source Hashing，源地址哈希；对用户来源IP做hash计算实现会话保持
- DH：Destination Hashing，目标地址哈希；对目标地址进行hash，常用于缓存服务器实现正向代理，负载均衡内网用户对外部服务器的请求

#### 动态方法：根据算法及各RS当前的负载状态(Overhead)进行调度；

- LC: least connections，最少连接;优先调度至连接数最少的机器； Overhead=Active*256+Inactive
- WLC: Weighted LC, 加权最少连接；Overhead=(Active*256+Inactive)/weight
- SED：Shortest Expections Delay，最短期望延迟，弥补WLC中非活动链接数为0导致权重大的服务器一直响应的问题；Overhead=(Active+1)*256/weight
- NQ：Never Queue，从不排对
- LBLC：Locality-Based LC，基于本地的LC(动态的DH算法)；损失了命中率，提高了均衡性
- LBLCR：LBLC with Replication，带复制功能的LBLC
