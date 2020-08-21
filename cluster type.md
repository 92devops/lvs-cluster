#### lvs 组成

从Linux内核版本2.6起，ip_vs code已经被整合进了内核中，因此，只要在编译内核的时候选择了ipvs的功能，您的Linux即能支持LVS。Linux 2.4.23以后的内核版本也整合了ip_vs code，但如果是更旧的内核版本，您得自己手动将ip_vs code整合进内核原码中，并重新编译内核方可使用lvs。ipvs 由用户空间的ipvsadm 命令行工具和内核空间的 ipvs 组成，支持基于TCP、UDP、SCTP、AH、EST、AH_EST等协议进行调度

- ipvsadm：用于管理集群服务及集群服务上的RS
- ipvs：根据用户定义的集群实现请求转发

#### lvs集群的专用术语：

Virtual Server：Director, Dispatcher（调度器）, Balancer（VS）

Real Server：后端提供真正服务的服务器(RS

Client IP：客户端IP(CIP

Virtual Server IP：LVS的前端IP(VIP

Director IP：LVS的后端IP(DIP)

Real Server IP：后端真正提供服务的服务器IP(RIP)

####  LVS工作原理：

LVS工作在内核的INPUT链上,请求的数据报文到达INPUT链时，如果用户请求的数据报文为本机则经用INPUT链进入用户空间。如果访问的目标IP和目标端口为IPVSADM定义的规则相匹配,则强行改变数据包的流向,送往POSTROUTING链,再送往后端集群组(转发到内部服务器类似于iptables的DNAT,但它不更改目标IP(也可以更改目标IP)

#### lvs 集群类型

##### lvs-nat(多目标的DNAT)

通过将请求报文中的目标地址和目标端口修改为挑选出的某RS的RIP和PORT实现转发

工作特性：

- DIP 和 RIP 在应该使用私有地址(隐藏后端服务器)，且必须处于同一个网络 RS 网关必须经由 DIP(保证响应保证必须经由 Director)
- 请求报文和响应报文都必须经由Director，较高负载下，Director容易成为系统瓶颈
- 支持端口映射
- VS必须是Linux，RS可以是任意OS；只要能提供相同的服务即可

##### lvs-dr(Direct Routing直接路由)

通过为请求报文的重新封闭一个MAC首部进行转发，源MAC是DIP所在接口的MAC，目标MAC是挑选出某RS的RIP所在接口的MAC地址；IP首部不会发生变化（CIP<-->VIP）

工作特性:

- 确保前端路径器将目标IP为VIP的请求报文发往Director:
  * 在路由器上静态绑定VIP和Director的MAC地址
  * 禁止RS响应VIP的ARP请求，禁止RS的VIP进行通告
    * arptables: 定义一个ARP的访问控制
    * 修改各RS的内核参数,并把VIP配置在特定的接口上实现,并禁止其响应发送ARP广播(通常将VIP配置在lo的子接口上)`arp_ignore`, `arp_announce`
- RS的RIP可以使用私有地址，也可以使用公网地址
- RS跟Director必须在同一物理网络(基于mac地址转发)；RS的网关必须不能指向DIP
- 请求报文必须由Directory调度，但响应报文必须不能经由Director
- 不支持端口映射
- RS可以使用大多数的OS

#### lvs-tun(tunnel隧道模型)

转发方式，不修改请求报文的IP首部（源IP为CIP，目标IP为VIP），而是原IP首部之外再封装一个IP首部（源IP为DIP，目标IP为挑选出的RS的RIP）

工作特性：

- RIP，DIP，VIP全得是公网地址；DIP也可以为私网地址
- RS网关不能指向也可能指向DIP
- 请求报文经由Director转发，但响应报文将直接发往CIP
- 不支持端口映射
- RS的OS必须支持隧道功能

#### lvs-fullnat

通过同时修改请求报文的源IP地址（CIP-->DIP）和目标IP地址（VIP-->RIP）进行转发

工作特性：

- VIP是公网地址，RIP和DIP是私网地址，且通常不在同一网络中，但需要经由路由器互相通信；
- RS收到的请求报文源IP为DIP，因此响应报文将直接响应给DIP
- 请求和响应报文都经由Director
- 支持端口映射
