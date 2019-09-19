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


## 竞赛规则
- 各参赛队伍应当在组委会的规定技术框架内进行公平竞争。不允许以任何方式入侵、攻击、滥用组委会服务器或基础设施。
- 禁止队伍间窥探、交换私钥、token和其它私密信息。
- 禁止冒用、顶替他人身份。禁止非法组队。
- 组委会有权禁止未预计的任何非正常比赛行为，有权禁止严重影响比赛平衡的漏洞利用行为。

## 开发细节-API
为了方便参赛队员在短时间内接入到区块链网络，组委会封装了所有在比赛中需要/能够使用的API，并提供了三种语言（Go、Python、Java）的SDK，使得选手可以专注于对DAG网络的理解和策略的制定上，而无需在有限的时间内关注接入细节。参赛队员并不需要在本机运行全节点或开发接入代码。

每个队伍都会被分配一个唯一的token，用于进行API的请求。该token用于限制用户进行大量恶意的攻击。
开发者只需要替换掉代码中token的内容即可，使用SDK发送http请求时token已经被处理，开发者无需关心。

详细的API接口如下（以Java为例）：
### queryNonce(String address)
返回对应账户地址的当前Nonce。在发送交易前，用户需要提供一个Nonce，以区别前后交易，避免双花。
即，如果本次交易需要使用的Nonce为6，如果这笔交易被成功验证，则下一笔交易的Nonce为7。
如果在同一个Sequencer中发送两笔交易，也需要遵守这个规则。
即使交易未被Sequencer确认，只要节点收到这笔交易（进入TxPool），queryNonce的结果会被更新为该笔交易中的Nonce值。
注意：**一个合法的交易，如果Nonce=n，则它必须有一个祖先交易（可以是非直接父交易），该祖先交易的Nonce为n-1，即单个地址发送的交易必须在图上能够被一条单一的路径串联，且Nonce呈递增关系。**
如果该交易最终被验证为非法，则Nonce会被回滚。

### sendTx(Transaction tx) 100次/分钟
向区块链网络发送一笔交易。该方法会自动处理交易的签名。
Transaction的详细内容如下：
- **parents**：根据DAG和比赛的要求，该笔交易需要选择1-2笔父交易，以形成合法的DAG结构。此处填入父交易的Hash值数组。
- **from**：自己的地址，可以通过account.Address()拿到。
- **to**：必须为null/None/nil，比赛环境不允许私下转账。
- **nonce**：该笔交易所使用的Nonce。该Nonce只能被用于一笔合法的交易上，按照交易数量递增1。
- **guarantee**：对这笔交易所下的赌注/保证金。
- **value**：必须为0。比赛环境不允许私下转账。

### queryBalance(String address)
返回对应账户地址的当前余额。

### queryTransaction(String hash)
查询指定Hash的某一笔交易

### querySequencerByHash(String hash)
查询指定Hash的某一笔Sequencer

### querySequencerByHeight(long height)
查询指定高度的某一笔Sequencer

### queryTxsByAddress(String address)
查询某一个地址的所有历史交易

### queryTxsByHeight(long height)
查询某一个高度上的所有交易

### queryAllTipsInPool() 5次/分钟
查询当前在交易池中所有没有被其它交易跟随的交易

### queryAllTxsInPool() 5次/分钟
查询当前在交易池中所有的交易

## 开发细节-MQ

为了使开发者能够快速收到整个网络的最新交易，组委会提供了Kafka消息队列设施，用于通知订阅者最新的交易事件。
消息队列会向开发者推送如下消息：
- 新的Sequencer产生：
```
{'type': 1, 'data': {'type': 1, 'hash': '0x97efc39c5f7eb8e319fff579a50afe0b69a463f80441e5635c37d073dfd3dc74', 'parents': ['0xedc86567ffaf12df14c39360a45bcf2126ca406eb2f3ea94d17ebf82e7685179'], 'from': '0x7349f7a6f622378d5fb0e2c16b9d4a3e5237c187', 'nonce': 7885, 'treasure': '1000', 'height': 7885, 'weight': 12925}}
```
- 新的（合法的但是未经确认的）Transaction产生：
```
{'type': 0, 'data': {'type': 0, 'hash': '0x6513196894db559ea4c4b673a21a96b370c66bd3ceec0abe29ea6ec34f203006', 'parents': ['0x97efc39c5f7eb8e319fff579a50afe0b69a463f80441e5635c37d073dfd3dc74'], 'from': '0xc4321fee1e29b13b042feab06dea55e7caf85948', 'to': '0x0000000000000000000000000000000000000000', 'nonce': 5040, 'guarantee': '100', 'value': '0', 'weight': 12926}}
```
通过解析该JSON字符串，我们可以获取到最新的交易，从而绑定上对应的交易发送逻辑。
一些可能的策略是：
- （收到Sequencer）-（立即发送交易，以Sequencer的Hash为parent）
- （收到Sequencer）-（等15秒）-（发送交易，以Sequencer和场上任何一个其它交易（如有）的Hash为parents）
- （收到Tx）-（发送交易，以Tx的Hash为parents）

策略的制定是灵活的，开发者可以根据比赛的进展、对手的策略等实时调整自己的发送策略。
