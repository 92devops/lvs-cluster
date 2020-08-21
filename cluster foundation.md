#### 集群的概念

将多台主机组织起来满足同一个需求的大型分布式机器群组称为集群

#### 	集群的类型

LB：Load Balancing, 负载均衡集群

- 硬件实现
  - F5 BIG-IP
  - Citrix Netscaler
  - A10 A10
  - Array
  - Redware

- 软件实现
  - lvs: Linux Virtual Server
  - haproxy
  - nginx
  - ats (apache traffice server)
  - perlbal

HA：High Availibilty, 高可用集群

- 软件实现
  - keepalived：vrrp协议
  - ais：
    - heartbeat
    - cman+rgmanager(RHCS)
    - corosync+pacemaker

HP：High Performance, 高性能集群

####
