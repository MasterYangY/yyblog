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
银河麒麟Kylin 等。

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

![](https://yyblog.win/assets/img/OB/OB21.png)

数据实时同步+快速切换+回滚预案
