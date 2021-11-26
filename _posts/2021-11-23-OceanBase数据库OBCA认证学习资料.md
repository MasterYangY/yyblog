---

layout: post
title: 'OceanBase数据库OBCA认证学习资料'
date: 2021-11-23
author: YY
cover: 'https://yyblog.win/assets/img/OB/oceanbase.png'
tags:  OceanBase
---
*  目录
{:toc}

# 1.分布式数据库与集中式数据库的差异

## 1.1 传统集中式数据库问题

- 1、基于单点高端硬件架构，成本高

- 2、无法横向扩展

## 1.2 分库分表方案--使用数据库中间件

- 1、应用侵入性问题

- 2、功能限制，不支持复杂SQL等

- 3、不保证数据一致性，无法保证失误ACID

## 1.3 分布式数据库特点（OceanBase特点）

- 1、采用Paxos协议，保持一致性

- 2、使用普通PC硬件，成本低

- 3、RPO=0，RTO<30s

- 4、数据存储和计算能力可横向扩展

# 2.OceanBase产品家族及基础概念

## 2.1 产品家族

![](https://yyblog.win/assets/img/OB/OB1.png)

- ODP 数据中间件
- OCP运维平台
- ODC开发平台
- OMS数据迁移平台

支持的CPU：Intel X86 系列CPU、海光（Hygon 7185）、海思（鲲鹏920）、飞腾（FT1500a、
FT2000）；
操作系统：CentOS、Red Hat、SUSE、Debian/Ubuntu、AliOS、中标麒麟NeoKylin、
银河麒麟Kylin 等。**不支持Windows**。

黑屏工具：OceanBase客户端，MySQL客户端

白屏工具：OceanBase云平台，OceanBase开发者中心

## 2.2 基本概念

#### 集群、Zone和OB Server

![](https://yyblog.win/assets/img/OB/OB2.png)

一个集群由多个zone组成，每份数据在各个zone上都有且只有一个副本，单zone故障不影响业务。

Zone在逻辑上就是给集群里的一批机器打上tag，同一个tag下的服务器就属于同一个zone。zone一般大于3，分散在不同地方备灾。

OB Server相对独立，有独立计算和存储引擎。

#### RootService总控服务（RS）

![](https://yyblog.win/assets/img/OB/OB3.png)

RS是OB的核心模块，管理整个集群，负责系统初始化，资源分配调度，全局DDL，集群数据合并等。每个Zone有一个RS服务，但是只有一个为**主**，其余为**备**。**RS无需额外部署，一般与OB server共用一台服务器**。

#### 租户

![](https://yyblog.win/assets/img/OB/OB4.png)

集群的多个服务器组成大资源池，系统给租户创建虚拟资源池以供使用。

租户的隔离策略：**内存物理隔离，CPU逻辑隔离，数据隔离**。系统租户保存系统表，ID一般在1000以内。**创建租户需要指定是MySQL模式合适Oracle模式**。

#### 资源池

![](https://yyblog.win/assets/img/OB/OB5.png)

UNIT描述了一组资源，每个UNIT都只属于一个租户。

一个租户在一个server上只能有一个UNIT。

# 3.OceanBase集群技术架构

## 3.1 Paxos协议与负载均衡

#### 数据分区与分区副本

![](https://yyblog.win/assets/img/OB/OB6.png)

数据分区：大表拆成不同分区，分区还能再拆成二级分区。分区的不同副本存在不同Zone中，只有一个主副本，其他为从副本。

根据数据到分区的映射关系不同，可以分为：

- **hash分区**
- **list分区（按列表）**
- **range分区（按范围）**

副本（Replica）由**日志**、**存储在内存的增量数据（MemTable）**、**硬盘上的静态数据（SSTable）**组成，副本可以分为分为

- **全能型部分（全量数据，最常用副本，参与投票）**
- **日志型副本（只有日志，参与投票，但不会变成主副本-）**
- **只读型副本（全量数据，只做listener，不属于paxos组，不参与投票）**

每个分区在一个zone里只能有一个全能型或日志型副本，但是可以有多个只读型副本。

#### 多副本一致性协议（Paxos协议）

![](https://yyblog.win/assets/img/OB/OB7.png)

Paxos组特点：

- OB的paxos组以分区为单位，细粒度小
- 组成员自动选出主从，无需人工干预
- 副本均匀分布在Zone内的机器上，避免单个Zone故障影响业务

#### 自动负载均衡和智能路由

![](https://yyblog.win/assets/img/OB/OB8.png)

主从副本被打散在各服务器中，使各服务器都能承载业务流量。

每台OB Server都可以独立执行SQL，自动访问其他机器数据，践行“首问责任制”和“最
多访问一次”的概念，避免对业务的侵入性。

#### 多副本同步Redo-Log保证数据持久化

![](https://yyblog.win/assets/img/OB/OB9.png)

写数据时，主副本所在服务器将Redo-Log发送到从副本所在机器进行落盘，当从副本返回落盘成功消息给主副本的数量足够满足多数派要求后，即可反馈应用操作成功，无需等待其他副本。

这种机制优点：

- **只要Redo-Log落盘即可，速度快**
- **Redo-Log日志同时同步给2个以上副本，只要多数派落盘成功即可，不用等所有副本都成功，响应快**

#### OB Proxy智能路由服务

![](https://yyblog.win/assets/img/OB/OB10.png)

OB Proxy能进轻量SQL解析，获取SQL中表的主副本所在的机器，多个OB Proxy 没有联系，可以组成F5/SLB负载均衡集群。**OB Proxy 是一个”无状态“的服务进程，不做数据持久化，不参与数据库引擎的计算任务**。

OB Proxy 不单独占用服务器，可以与OB Server共用服务器。

#### Primary Zone

![](https://yyblog.win/assets/img/OB/OB11.png)

租户配置Primary Zone，可以将业务汇聚到特定Zone，逗号两侧同优先级，分号左侧优先。

**配置优先级：租户<数据库<表**，无特别指定，则继承上级Primary Zone设置，数据库继承租户，表继承数据库，也可单独设置库或表的优先级。

#### Table Group

![](https://yyblog.win/assets/img/OB/OB12.png)

Table Group将多个多个表分区方式完全相同（分区类型、分区键个数、分区数量等），可以再逻辑上将这些表归属到同一个Table Group中。

同一个Table Group中的表，分区ID相同的分区，在同一个OB Server上。

## 3.2 数据可靠和高可用

OceanBase的**RPO=0，RTO<30秒**，意味着当少数派故障时，OceanBase 能够在30 秒内恢复业务，且不会丢失任何数据。

OB server进程异常后的处理策略：

- 小于server_permanent_offline_time，进程终止时间不长，暂不做处理

- 大于server_permanent_offline_time，“临时下线”，从其他zone的主副本中，将缺失的数据复制到本zone内剩余的机器上，以维持副本个数

## 3.3 动态扩容缩容

扩容缩容步骤：

购买资源 -> 追平数据 -> 切换服务 -> 缩容 -> 退回资源

扩容步骤：

1. 为每个zone添加新的物理机器
2. 在每台新添加的机器上，以正确的方式启动observer服务
3. 为每台新添加的observer进程执行alter system ad server;命令，将observer服务添加到集群中
4. 执行alter resource pool poolname unit_num=xx;扩充资源池中的unit个数
5. oceanbase自动启动“rebalance”过程，将部分数据从旧的unit在线复制到新的unit上。
6. 每个分区复制完成后，OceanBase自动将服务切换到新的unit上，并删除旧unit中分区上的数据

> 确定是否复制完成？
> 查看`__all_virtual_sys_task_status`表，是否有`comment like ‘%partition migration%’`的任务


缩容步骤：

1. 执行alter resource pool poolname unit_num=xx;缩减资源池中的unit个数
2. OceanBase自动启动“rebalance”过程，将一部分数据从待下线机器上的unit在线复制到同zone内其它机器的unit上
3. 每个分区的数据复制完成后，OceanBase自动将服务切换到新的unit上，之后删除待下线机器的unit分区上的数据
4. 为每台要下线的机器执行alter system delete server；命令完成机器下线
5. 手动终止已下线机器的observer进程，执行关机等操作

## 3.4 分布式事务、MVCC、事务隔离级别

#### ACID

**A（Atomicity）原子性**，使用**两阶段提交**保证原子性。

- 第一阶段：投票阶段，协调者向所有参与者提交发送prepare请求与事务内容。

- 第二阶段：所有参与者都回复OK则协调者发送提交请求，参与者完成后返回给协调者。若有任何参与者回复no，则回滚。

两阶段提交方案中，参与者和协调者都是OB Server。

**C（Consistency）一致性**，通过**保证主键唯一、全局快照**等技术保证一致性。

**I（Isolation）隔离性**，采用**MVCC（所版本并发控制）**进行并发控制，实现**read-committed**的隔离级别；所有修改的行加互斥锁，实现写 - 写互斥；读操作读取特定快照版本的数据，读写互不阻塞。

- 修改数据时：获取行锁再修改数据，食实物未提交前先根据版本号区分数据，提交后只有新数据。
- 读取数据时：无需加锁，也无需关注数据是否加锁，先获取版本号，再找小于等于当前版本号的已提交数据。

OceanBase支持的事务隔离级别：**read-committed**（默认）和**serializable**。

**D（Durability）持久性**，使用**Paxos协议**，通过在多副本之间**同步Redo-Log**和**Redo-Log 落盘**来确保数据的持久性。

## 3.5 SQL引擎和存储引擎

#### 引擎兼容

- MySQL 5.6语法全兼容；兼容MySQL通信协议，MySQL应用可直接迁移至OceanBase 

- 兼容Oracle 11g语法，支持90%的Oracle数据类型和内置函数，支持分布式执行的存储过程（PL/SQL）。

#### 准“内存数据库”+LMSTree存储，避免随机写和写放大

![](https://yyblog.win/assets/img/OB/OB13.png)

OceanBase 把内存分为了两块，默认50%，一块是**MemTable**，用于写；一块是**热点缓存**，用于读。

- 写：先写到MemTable内存中，**MemTable到达阈值后启动转储**，**转储次数到达阈值或定时后启动合并**，同时Redo-Log实时落盘，保证数据持久性。
- 读：从**MemTable内存**，**转储数据**，和**基线数据SS Table**中都读取数据。

#### LSMTree存储方式

![](https://yyblog.win/assets/img/OB/OB14.png)

存储顺序

**memstore** 

-> **mini freeze（dump操作）** 

-> **minor freeze（转储）** 

->**major freeze（合并）** 

-> **基线数据sstable**
多个`mini freeze`数据会**异步**合并，多个`minor freeze `会**实时**合并

#### LSMTree数据压缩

两次压缩：

- 第一次是**encoding**，会使用字典、RLE 等算法对数据做瘦身。
- 第二次是通用压缩，使用**lz4** 等压缩算法对encoding 之后的数据再做一次瘦身，OceanBase 支持snappy、lz4、lzo，zstd 等压缩算法。

#### 备份恢复

**支持全量备份和增量备份**：直接对存储层基线数据做全量备份，通过redo-log实现增量备份。

可以在线实时进行全量及增量备份，对业务无影响。

备份恢复最小粒度为**租户**。

全局范围内保证数据一致性。

支持数据库上的任何操作。

支持多种备份介质：**普通NFS、阿里云对象存储（OSS）**。

性能：备份速度达到网卡上限1G/s，恢复速度500MB/s。

# 4.参数和变量

参数分为动态生效和重启生效，多数为动态生效。

参数级别有集群级和**租户级**，多数为集群级。如果同时存在集群级和租户级参数，则集群参数覆盖租户参数。

系统租户可以查看和设置所有其他租户参数，普通租户只能设置自己租户参数。

##  4.1 参数permanent

#### 集群参数

![](https://yyblog.win/assets/img/OB/OB15.png)

#### 常用OB系统参数（合并相关）

![](https://yyblog.win/assets/img/OB/OB16.png)

#### 常用OB系统参数（syslog相关）

![](https://yyblog.win/assets/img/OB/OB17.png)

#### 常用OB系统参数（内存相关）

![](https://yyblog.win/assets/img/OB/OB18.png)

#### 常用OB系统参数（其他）

![](https://yyblog.win/assets/img/OB/OB19.png)

## 4.2 变量Variables

#### 变量分类
会话变量**session**：当前会话生效。

全局变量**global（租户级）**：租户级生效，当前会话不生效，需要重新建立会话。

#### 查看变量
```SQL
show variables;
show variables like '%%'
```
#### 修改变量
```SQL
set @@session.<name>=<value>；
set @@global.<name>=<value>；
```

#### 常见OB系统变量variables

![](https://yyblog.win/assets/img/OB/OB20.png)

# 5.OCP、ODC和OMS工具

## 5.1 OCP

OCP（Oceanbase Cloud Platform，OceanBase云平台）是企业级数据库**管理**平台，也就是运维工具。核心功能：

- 集群管理：提供全生命周期管理，包括安装、运维、性能监控、配置、升级和删除等功能。

- 主机管理：提供添加主机、删除主机、主机关键信息显示等功能。

- 租户管理：租户的创建、租户结构拓扑图、性能监控、会话管理和参数管理等。

- 告警管理：支持集群、租户、主机等不同维度的告警。系统基于告警规则生成告警。

- 备份恢复管理：支持对OceanBase集群和租户级别进行全量备份、增量备份、Redo-Log备份、完全恢复、不完全恢复等功能。

- 用户及权限管理：通过对用户和角色的管理确保系统安全。

## 5.2 ODC

ODC（Oceanbase Developer Center，OceanBase开发者中心）是企业级数据库**开发**平台。核心功能：

- 特性分区支持：支持OceanBase MySQL和Oracle模式下完整的分区类型。
- 多种文件格式数据导入导出。
- WebSQL
- 变量可视化
- 资源性能查看
- 数据库对象可视化

## 5.3 OMS

OMS（OceanBase Migration Service，OceanBase迁移服务）

![](https://yyblog.win/assets/img/OB/OB21.png)

核心功能：

- 多种类型数据源：支持Oracle、MySQL、DB2、OceanBase数据库到OceanBase的全量迁移和增量实时数据同步。
- 兼容性评估和改造：给出异构数据库的兼容性评估和改造建议。
- 一站式交互。
- 多重数据校验。

OMS平滑去O方案

![](https://yyblog.win/assets/img/OB/OB22.png)

数据实时同步+快速切换+回滚预案

# 6.题库

【判断题】分库分表的架构虽然解决了集中式数据库的扩展性问题，但也带来了新的问题(不支持复杂SQL， 较难保证分布式事务的ACID等) 。T

【判断题】TPC-C就是一个跑分测试， 没有什么规则限制,只要能跑高分就行 F

【判断题】Ocean Base数据库是在阿里和蚂蚁内部孵化了10年后才逐步推广到外部市场的。T

【判断题】Ocean Base数据库是基于开源数据库的再发行产品。 F

【判断题】Ocean Base已发布到阿里云公有云及专有云中。 T

【判断题】Ocean Base只支持X 86架构的CPU， 不支持国产CPU(如鲲鹏、海光、飞腾等) F

【判断题】Zone是个逻辑概念， 是给集群内的一批机器打上同一个tag， 属于同一个tag的服务器归属一个Zone。T

【判断题】Zone可以对应不同的城市， 或者一个城市的不同机房， 或者一个机房的不同机架。 T

【判断题】租户的资源池一旦创建完成，就不可改变。 F

【判断题】分区的副本只包含硬盘上的静态数据(S STable) ， 不包括Mem Table数据和日志数据。 F

【判断题】主副本只能打散到所有Zone内， 不能聚焦到一个Zone内 F

【判断题】每台OBServer是相对独立的， 都有自己独立的SQL引擎， 如果应用需要的数据不在当前OBServer上， 该OB<br>Server将协调其他OBServer的数据， 统一反馈给应用， 这个过程对应用是透明的。 T

【判断题】主副本通过同步Redo-Log日志的方式实现可靠性， 主副本需要收到所有从副本落盘成功的消息后才能响应应用。 F

【判断题】企业在一个城市有2个机房， 将2个Zone部署到1个机房中， 将另一个Zone部署到另一个机房中， 可以提供机房级的容灾。 F

【判断题】 Ocean Base可以支持在一个集群中同时支持MySQL租户和Oracle租户。 T

【判断题】使用Explain命令查看SQL执行计划时， SQL也会真正执行。 F

【判断题】合井必须依赖Ocean Base自动完成， 无法手工启动合并。 F

【判断题】Ocean Base的数据在磁盘中按主键有序排列。 T

【判断题】会话变量只对当前会话生效，不影响该租户下的其他会话。 T

【判断题】Global级(租户级) 变量修改后， 对当前已经打开的session也依然生效。 F

【判断题】如果同时存在集群级别参数和租户级别参数，那么集群级别参数将覆盖租户级别参数。 T

 

【多选题】传统的集中式关系型数据库面临哪些挑战? AC

A：成本高：运行在高端服务器、小型机、高端存储等专有硬件上；

B：生态欠缺：文档、培训、应用等都不足；

C：扩展性差：无法摆脱单机的架构，只能纵向扩展，无法横向扩展；

D：性能差：任何时候，传统集中式数据库的性能都比分布式数据库较差；

 

【多选题】Ocean Base的核心特性有哪些? ABCD

A：高扩展，可以使用普通的PC服务器进行横向扩展；

B：高性能，峰值峰值6，100万次/秒，单表最大3，200亿行；

C：高可用， 通过Paxos协议保证强一致性， RPO=0， R TO<30秒；

D：高兼容， 支持MySQL及Oracle两种模式， 降低业务迁移改造成本；

E：高成本，使用小型机、高端存储等专有硬件；

 

【多选题】Ocean Base主要有哪些产品组成? ABCD

A：数据库内核：提供SQL引擎及存储引擎， 同时兼容MySQL和Oracle模式； 使用Paxos协议确保高可用性；

B：OCP云管理平台：给管理员提供的管理工具， 提供集群管理、Zone管理、租户管理等功能；

C：OMS数据迁移工具：提供基线数据和增量数据的同步功能， 可以从数据仓库订阅数据链路、从异构数据库迁移数据；

D：ODC开发者中心：提供数据库日常开发、SQL诊断、会话管理及数据导入导出能功能。

 

【多选题】Ocean Base支持哪些事务隔离级别 BC

A：脏读 

B：Read-Committed 

C：Serializable

 

【多选题】以下对OB Proxy的描述是正确的  AD

A：OB Proxy位于应用和OBServer之间， 将应用的请求路由到合适的OBServer；

B：OB Proxy需要部署到一台独立的服务器上， 以保证其性能要求；

C：OB Proxy参与数据库引擎的计算任务以及事务处理；

D：OB Proxy是一个“无状态”的服务进程， 不做数据持久化

 

【多选题】Ocean Base备份恢复业务支持哪些存储介质  AD

A：NFS B：IP-SAN C：FC-SAN D：阿里云OSS

 

【多选题】参数有哪两个级别?  AD

A：集群级

B：Zone级

C：OBServer级

D：租户级

 

【单选题】Ocean Base是一个什么类型的数据库  C

A：集中式数据库；

B：No SQL数据库；

C：分布式关系型数据库；

 

【单选题】Ocean Base是一个集群， 以下哪个组件管理整个集群， 支持全局DDL、集群数据合并等功能。  B

A：OB Proxy

B：Root Service总控服务

C：OCP管理平台 

D：ODC开发者中心

 

【单选题】Ocean Base集群可以同时支持MySQL和Oracle的租户， 哪个黑屏工具可以连接到Oracle租户  A

A：Ocean Base客户端；

B：标准MySQL客户端

 

【单选题】Ocean Base不支持什么操作系统  B

A：CentOS； 

B：Windows 

C：中标麒麟

D：银河麒麟

 

【单选题】如果一个Ocean Base集群有3个Zone， 每个Zone有5台OBSer er。那么一个分区有几份副本呢?  B

A：10 B：3 C：6 D：5

 

【单选题】如果一个集群有3个Zone， 每个Zone有5台OBServer。一个租户对应的资源池的Unit eNum=3， 最终该集群有多少个服务器中有该租户的资源单元呢?  B

A： 15 B：9 C：45 D：30

 

【单选题】Ocean Base是以() 为单位组建Paxos协议组。  D

A：租户 B：数据库 C：表 D：分区

 

【单选题】以下关于Ocean Base扩容和缩容描述正确的是。  C

A：需要管理员停止业务 

B：需要业务做一定的修改

C：支持动态扩容和缩容，对业务无感知

 

【单选题】Ocean Base使用两阶段提交协议保证事务的原子性， 在两阶段提交协议中， 谁是协调者呢?  B

A：OB Proxy 

B：OBServer

C：Root Service总控服务

D：OCP云管理平台

 

【单选题】Ocean Base使用哪种技术解决了读写互斥的问题。  A

A：MVCC

B：Paxos协议

C：全局快照

D：互斥锁

 

单选题】使用JDBC连接Oracle租户时， 需要使用哪种JDBC驱动。  C

A：MySQL标准的JDBC驱动

B：Oracle标准的JDBC驱动

C：Ocean Base自己开发的JDBC驱动

 

【单选题】为了达到更好的压缩效果， Ocean Base一般会进行进行几次压缩  B

A：1次 B：2次 C：3次 D：4次

 

【单选题】mini freeze是简单的dump操作， 多个mini freeze的数据会(  )合并； 多个minor freeze会(  ) 合并， 但不会和S STable合并。  B

A：实时、异步

B：异步、实时

C：实时、离散

D：离散、实时

 

【单选题】 Alter system命令可以修改集群参数和租户参数， 如该命令指定Zone或者OBServer， 最多可以同时指定几个?  A

A：1个 B：2个C：3个D：4个

 

【单选题】通过哪个命令可以查询参数的属性。 A

A：show parameters like'%<pattern>%'；

B：alter system set<name>=<value>；

C：show variables like'%<pattern>%'；

D：set@@global.<name>=<value>

 

【单选题】以下哪个组件提供图形化的管理界面，支持集群管理、租户管理、监控告警等功能?  B

A：ODC开发者中心 

B：OCP云管理平台

C：OB Proxy 

D：OBServer

 

 

判断：

1.	一个租户在同一个 Server 上可以有一个或多个资源单元 UNIT  	正确

2.创建资源单元仅仅指定  CPU、MEMORY 参数即可，无需指定 OPS、DISK_SIZE、SESSION_NUM参数  错误

3.OCEANBASE 在少数副本不可用的情况下，可以实现 RPO=0,RTO<30 秒   正确

4.Zone 可以对应不同的城市，或者一个城市的不同机房、或者一个机房的不同机架，以实现不同级别的容灾  正确

5.主副本只能打散到所有 Zone 内，实现访问流量的负载均衡，不能将主副本聚焦到一个Zone内。  错误

6.扩容服务器加入集群后，集群会基于负载均衡的策略，将主副本及从副本迁移到扩容服务器中，以实现整体的负载均衡   正确

7.租户逻辑上类似传统数据库实例，创建完成后，每个租户都拥有自己的专属进程   正确

8.OceanBase 的 Paxos 协议，不同于传统的主备库或者双选方案，可以彻底规避在容灾场景下的脑裂问题（也就是同时又两个主数据库的场景）   正确

9.修改资源池可以实现租户的另一种扩容/缩容的方式，比如在每个 zone 中增加/减少节点数量，可以通过修改资源池的 unit_num 来实现	  正确

10.创建租户时，需要指定租户类型为 Oracle 租户或者 MYSQL 租户，以满足不同开发者的需求。  正确

11.同一个资源单元定义 unit cofig(比如 2C8G，或者 4C16G 等)，可以被多个资源池使用。  错误

 

多选：

 

1.OMS 实时同步工具是异构数据库迁移到 OceanBase 的利器，OMS 支持哪些功： BCDE

A:支持会话管理和系统全局变量的可视化修改，用户记忆变量的难度

B:支持多种类型数据源，支持包括 Oracle、MYSQL、DB2、OceanBase 等数据库到

OceanBase 的全量迁移和增量实时数据同步

C:一站式交互，数据迁移全生命周期管理，数据迁移的创建、配置和监控都在管控界面上连贯操作完成，交互简便

D：兼容性评估和改造：异构数据迁移 OceanBase 的对象兼容性评估和改写建议，极大降低业务迁移的门槛和业务改造的难度。

E:多重数据校验：提供多种方式校验的保护。要更加全面、省时、高效地保证数据质量

 

2.关于 OceanBase 的 Zone，以下说法正确的是CDEF 

A:每个 Zone 可以包含一个分区的多个副本

B:不同 Zone 一定要部署在不同机房

C:一个分区的多个副本应分布在不同的 Zone 中，每个 Zone 有且只有分区的一个全功能副本

D:Available Zone 的含义是可用区，通常指一个机房

E:一个 OceanBase 集群由若干个 Zone 组成

F:一个 Zone 包括若干物理服务器

3.关于 OceanBase 的系统参数的生效范围，以下说法正确的是： ABC 

A:可以在某台 OBServer 生效

B:可以在某个 Zone 生效

C:可以在集群范围生效

D:可以在某个 Region 生效

4.随着业务不断发展，原有租户的资源无法满足业务需要，有哪些扩容方式？ BC 

​	A:无法对租户进行扩容，需要创建一个新的租户满足业务需要

​	B:调整资源池中，资源单元（resource unit）的数量，如原数量是 1，可以增加为 2

​	C:调整资源池里的资源单元（resource unit）的规则，比如之前规格是 2C8G,可以调整为 4C16G

5.RootService 总控服务提供资源分配及调度功能，主要包括哪些功能： ABCD

 A：分区及副本管理

B: 动态负载均衡

C:SQL 引擎

D:扩容和缩容

6.关于 OceanBase 的修改系统参数命令 ALTER SYSTEM SET XX=’YY’，以下说法正确的是：  ABCE

A:如果不要任何条件，则会返回错误；

B:可以修改该 Parameter 在某个 zone 上的值

C:可以修改该 Parameter 在某台具体的 OBServer 上的值

D:如果不带任何条件，则修改所有 OBServer 的值

E:可以修改 Parameter 在某个 Region 的值

7.关于 OceanBase 的分区 Partition，以下说法正确的是：AB

A:数据表根据分区规则，拆分成多个分区，每个分区包括表中的若干行记录

B：每个分区，还可以用不同的分区维度再进行分区，叫做二级分区 C:OceanBase 只支持一级分区，不支持二级分区

D: OceanBase 的分区是数据迁移的最小单元，也是高可用切换的最小单元

E:OceanBase 支持表的自动分区分裂

8.关于租户的扩容方式，以下说法正确的是：  AB

A：租户扩容，可先通过添加服务节点，完成集群扩容，再通过增加资源单元的个数完成租户扩容

B:如果集群和节点资源足够，可以直接修改租户资源池相关的资源单元规格大小，进行扩容

C:OceanBase 是分布式集群具有横向扩展的能力，租户扩容仅仅需要添加阶段即可，无需扩容租户的资源单元

D:租户无法进行扩容，如果资源无法满足需求，需要重新建立更大资源池的租户。

 

9.系统管理员可以根据业务需要创建不同的租户，租户具有哪些特性 ABCD 

A:有自己独立的系统变量

B:有独立的 information_schema 等系统数据库

C:可以创建自己的用户

D:可以创建数据库，表等所有对象

11.关于 OceanBase 的应用日志级别，以下说法正确的是： CDE 

A:warn 警告，用于记录严重错误，需要立即处理

B：info 提示，用户记录系统运行的当前状态，该信息为错误信息

C:ERROR 严重错误，用于记录系统的故障信息，且必须进行故障排除，否则系统不可用

D: info 提示，用户记录系统运行的当前状态，该信息为正常信息

E:warn 警告，用于记录可能会出现的潜在错误

 

12.分区数据一般有多份副本，OceanBase 的 副本有什么类型：ACD

A:全能型	B 只写型 C:日志型 D:只读型	

 

13.OceanBase 开发者中心 ODC 是为 OceanBase 数据库量身打造的企业数据库开发平台，主要支持哪些功能 ABCDE 

A:提供引导式创建和可视化修改各类数据库对象的服务

B:支持多种文件格式的导入和导出

C:通过 WebSQL 技术为开发人员提供 SQL 语法高亮、格式化、只能提示等贴心特性、支持 PL 对象及匿名快的编译、运行调试

D:实时管控数据库会话访问，支持查看和终止会话，且提供 SQL 执行计划分析和 SQL 调优指导服务

E:支持会话变量和系统全局变量的可视化修改，降低用户记忆变量的难度

 

14.关于 OceanBase 的租户权限管理,以下说法正确的是：AB

A:任何租户（，不论是系统租户还是普通租户）下的用户不能跨租户访问其他普通租户下的用户数据

B:只有系统租户下的管理员用户才有集群管理的权限，执行系统管理操作，如创建/删除普通租户。设置系统配置参数，开启每日合并操作

C:系统租户下的管理员用户可以访问其他普通租户的用户数据

D:系统租户下的管理员用户可以给其他普通租户的用户进行授权，使得普通租户的用户拥有系统管理员的权限

 

 

15.关于 OCP 的告警功能，下列说法正确的是：  ABCDEF

​	A:OCP 告警依赖专有云底座

​	B:可以查看告警列表

​	C:可以调整告警阈值

​	D:不支持用户修改告警阈值

​	E:可以自定义告警发送对象

​	F:可以调整告警开关，确定哪些项需要监控

 

16.关于 OceanBase 实物引擎的 MVCC 多版本并发控制，以下说法正确的是： ACD 

A: 读操作读取特定快照版本的已提交数据

B：写会阻塞读操作

C: 所有修改的行加互斥锁、实现写-写互斥

D: 读写互不阻塞

17.OceanBase 支持哪些分区方式的分区表 ABD 

A：Range 

B：Hash 

C：Datetime 

D：list

19.	以下哪个描述不是 OceanBase 的架构特点：	中心管控

\20. 租户创建完成后，可以使用黑屏客户端连接数据库，除了指定数据库的 IP、端口号、用户名、密码等信息外，OceanBase 一般用户名使用什么格式 

用户名@租户名  例如 root@sys

21.	建立 table group 的主要目的是：	减少跨机分布式事物

\22. OceanBase 产品的数据库内核是什么  完全自主研发

23.当应用向数据库写数据时，默认会访问主副本，此次主副本会同步（）到从副本，保证数据的高可用 D

A:undo-log 日志

B:系统日志

C:心跳消息

D：redo-log 日志

 

24.以下哪个组件提供图形化的管理界面，支持集群管理、租户管理、监控警告等功能。  OCP云管理平台

25.部署 OceanBase 集群时，各个 OBServer 的 RPC 允许的时钟偏差最大是多少	100毫秒

26.如果一个 OceanBase 集群由 5 个 Zone，每个 Zone 有 10 台 OB Server，那么一个分区最多有几份全功能型副本  5个

27.	Linux 系统一般用什么用户来部署 OceanBase	ADMIN

28.	OceanBase 服务器要求使用的磁盘类型	: SSD固态磁盘

29.	假设OceanBase有3个Zone,其中2个Zone部署在一个城市的两个机房中，另外一个Zone部署在另外一个城市的一个机房中。如果同城的一个机房宕机，下面说法正确的是？   强一致同步延迟不变

30.	Major_freeze_duty_time 设置为 02:00 意味着什么	每日凌晨两点，系统自动发起一次内存冻结操作

31.关于 OceanBase 事物引擎一致性特点，描述正确的是：保证主键唯一等一致性约束

32.关于 OceanBase 资源隔离，以下说法正确的是   OceanBase采用租户隔离

33.管理员通过哪条命令创建资源池	create resource pool

34.OceanBase 是靠哪种基础架构实现写入高性能的	 LSM-TTREE

35.如果一个集群有 3 个 Zone，每个 Zone 有 5 台 OBServer，一个租户对应的资源池的 Unit Num=4,最终该集群有多少个服务器中有该租户的资源单元呢。   3*4=12 个
