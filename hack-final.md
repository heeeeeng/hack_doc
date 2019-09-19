#第二届众安黑客松大赛区块链赛道

内容简介

##背景知识

本次开发大赛将全程基于 AnnChain 的开源项目 OG 进行开发。选手需要了解 OG 的基本结构以及交易在 OG 上的确认方式。

#### DAG 结构的账本

在传统区块链结构中，交易信息会被打包进一个区块，区块之间以链式连接，形成一个区块链式的账本。

![区块链式结构](./区块链式结构.png)

**区块链特点 暂缺**

与之相对的，Dag (Directed acyclic graph) 有向五环图结构的账本则是将所有的区块打散，使交易与交易之间直接相连。通过交易来确认交易，保障交易哈希的不可篡改性。

![dag结构](./dag结构.png)

对比区块链结构的账本，dag能拥有更大的灵活性和可扩展性。不会因为矿工出块过慢卡住整个网络从而无法发送交易。

**更多的DAG特点 暂缺**

#### OG

tx结构

seq

og的特点，tip，tx_pool, dag, 

**更多的OG特点暂缺**

##规则

Dag的图结构使得交易之间存在一种直接的连接关系。在OG中，这种连接关系保证了交易hash间的关联。那么能否利用这种关联关系来增加区块链系统的可玩性呢？

有基于此，在本次黑客松中我们对这种交易间的父子关系增加了一个特殊的效果：

**前序交易parent会被后序交易child抢走部分资金**

![抢劫简单示例图](/Users/heeeeng/资料/众安/2019黑客松/决赛/文档/抢劫简单示例图.png)

如图所示，如果 `Tx03` 的 parents 为 `Tx01` 和`Tx02`，则 `Tx03` 会抢走它 parents 交易`Tx01` 和 `Tx02`的部分资金。

围绕这个规则，我们重新开发了OG并制定了一整套游戏玩法：

####抢劫规则

1. 每笔TX增加一个特殊字段 `guarantee`。`guarantee`你可以理解为保证金
2. seq会收尾所有交易，抢走60%
3. 

####空投规则

1. 每个seq会有空投，空投会被后续children交易100%抢走
2. 空投金额变化
3. 官方定时低保



**例子**



#### 评判标准

每队伍分配一个private key，内有一部分启动资金。

比赛结束时余额最多的队伍获得胜利。



##开发细节

- 每个队伍会分配一个token，用于操作sdk。
- 部分api会有请求限制，每分钟只能请求固定数量（列出详细API）
- 提供 GO / PYTHON / JAVA 的SDK
- 具体开发流程示例（伪代码）
