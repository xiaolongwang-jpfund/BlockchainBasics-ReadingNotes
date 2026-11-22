# 第19章：选择一份交易历史

## 本章核心问题

由于网络消息可能延迟、丢失或乱序，各节点可能暂时拥有不同的区块和交易历史。本章要回答：**当出现多条互相冲突的链时，节点如何选出共同采用的交易历史？**
作者用公园里人们踩出的小路作比喻：每个人的独立行动，大家的选择逐渐形成一条共同路径。这类没有中央协调者、由参与者共同达成的协议叫**分布式共识（distributed consensus）**。

## 必须掌握的区块链逻辑

1. **分叉如何出现**。
   两个节点可能在相近的时间分别创建新区块；消息传播又有延迟，因此不同节点可能先看到不同区块，形成多个分支。

2. **最长链标准**
   **longest chain criterion**:选择包含区块数最多的路径。若两条路径一样长，这个标准暂时不能明确决定选哪条。

3. **最重链标准**
   **Heaviest chain criterion**：当区块难度不同，不能只数区块；应比较**路径累计的工作量**。在本章模型中，路径上各区块的难度总和称为“**权重（weight）**，节点选择权重最高的链。

记忆区别：

- longest：数区块；
- heaviest：比累计权重

4. **未被选中的分支如何处理**
   不属于最终选定路径的区块可能成为**未被主链采用的区块即“孤块（orphan block）”**。其中有效且不冲突的交易可重新处理，之后可能再次被收录。相关奖励也可能被追回。

5. **共同主干与最终一致性**
   分叉可能暂时存在，但随着新区块不断添加，节点共同认可的主干会变长。某区块在权威链中越深，通常越不容易被后续变化推翻，大家对它属于该链也越有把握。这个随时间逐渐增强的一致状态称为**最终一致性（eventual consistency）**。

6. **工作量证明的作用**
   **工作量证明（proof of work，PoW）**，让创建新区块需要计算成本，也让节点被选中创建区块带有随机性。成本和不可预测性使攻击更难规划、代价更高。若攻击者要改写已被接受的历史，就必须重做相关工作，并追赶诚实节点的链。

7. **安全假设与51%攻击**

- **安全假设**：本章的 PoW 模型依赖“诚实节点掌握多数计算能力”这一假设。
- **51%攻击（51% attack）**：指攻击者试图获得或控制多数投票权/计算能力，以改变大家采用的交易历史。
  随机选中出块节点、攻击成本和诚实多数是本章解释系统抗操纵能力的关键。

**范围提醒**：这是本书为解释 PoW 链而建立的模型。不同区块链可能使用不同共识机制、链选择规划和奖励设计。

## 必须掌握的英语句型与结构

| 句型                                 | 用法          | 本章语境示例                                                                                                  |
| ---------------------------------- | ----------- | ------------------------------------------------------------------------------------------------------- |
| The goal is to + 动词原形              | 目标是……       | The goal is maintain a single history. 目标是维持一份统一的历史。                                                    |
| Due to + 名词，主句                     | 由于……        | Due to message delays, nodes may disagree. 由于消息延迟，节点可能意见不同。                                             |
| The idea of + V-ing … has led to … | ……这一想法引出了…… | The idea of selecting a history … has led to two criteria. 选择交易历史的想法引出了两条标准。                            |
| When … , 主句                        | 当……时，……     | When difficulty levels differ, the longest path may not represent the most work. 难度等级不同时，最长路径未必代表最大工作量。 |
| not necessarily …                  | 未必……，不一定……  | The longest path is not necessarily the heaviest. 最长路径未必是最重的。                                           |
| The + 比较级 …, the + 比较级 …           | 越……，就越……    | The deeper a block is, the less likely it is to be abandoned. 区块越深，就越不容易被放弃。                            |
| As soon as … , 主句                  | 一旦……，就……    | As soon as the speed competition is won, the quality competition begins. 速度竞赛一旦获胜，质量竞赛就开始。              |
| be free to + 动词原形                  | 可以自行……      | Nodes are free to decide which branch to extend. 节点可以自行决定延长哪条分支。                                        |
| make + 宾语 + 形容词                    | 使……变得……     | PoW makes attempts to manipulate the history more expensive. PoW 使得篡改历史的尝试成本更高。                         |
| by + V-ing                         | 通过做……；以……方式 | Nodes reach agreement by independently selecting a history. 节点通过独立选择历史来达成一致。                            |
| the fact that + 从句                 | ……这一事实      | The fact that certainty increases … is called eventual consistency. 确定性逐渐提高这一事实被称为最终一致性。                |
| in the face of + 名词                | 面对……时       | … resolve conflicts in the face of message delays. 面对消息延迟解决冲突。                                          |
| without rsorting to + 名词           | 不诉诸/不采用……   | … without resorting to a centralized solution. 不采用中心化方案。                                                |
| one after the other                | 一个接一个地，依次   | Peers submit blocks one after the other. 节点依次提交区块。                                                      |

## 本章高频短语与固定搭配

| 表达                                               | 含义                 |
| ------------------------------------------------ | ------------------ |
| reach consensus / reach agreement                | 达成共识 / 达成一致        |
| resolve conflicts among …                        | 解决……之间的冲突          |
| maintain a history                               | 维护一份历史记录           |
| select a version of the transaction history      | 选择一个交易历史的版本        |
| the longest criterion / heaviest chain criterion | 最长链标准 / 最重链标准      |
| aggregated computational effort                  | 累计计算工作量 ｜          |
| ｜ be part of the authoritative chain             | 是权威链的一部份           |
| extend a branch / chain                          | 延长一条分支 / 链         |
| refer to a block as its predecessor              | 以前一个区块作为一般用对象 / 前驱 |
| be removed from the blockchain                   | 从区块链中移除            |
| be reprocessed                                   | 被重新处理              |
| reclaim reward                                   | 收回奖励               |
| be likely / less likely to be abandonded         | 可能 / 不大可能被放弃       |
| be based on …                                    | 基于……               |
| be influenced by …                               | 受……影响              |
| incur costs                                      | 产生成本、承担代价          |
| receive a reward                                 | 获得奖励               |
| the majority of computational power              | 大部分计算能力 |
| manipulate the transaction history | 操纵 / 篡改交易历史 |

## 术语速记

- **consensus** /kənˈsensəs/: 共识、一致意见
- **distributed consensus** /dɪˈstrɪbjətɪd kənˈsensəs/: 分布式共识
- **authoritative(/əˈθɔːrəˌteɪtɪv/) chain/history**: 权威链 / 权威历史
- **orphan block**/ˈɔːrfən blɑːk/：孤块，即未被最重选定链采用的区块
- **weight**/weɪt/：（本章中的）权重，指累计难度
- **eventual consistency**/ɪˈventʃuəl kənˈsɪstənsi/：最终一致性
- **manipulation**/məˌnɪpjəˈleɪʃən/：操纵、篡改
- **computational effort**/ˌkɑːmpjuˈteɪʃənəl ˈefərt/：计算工作量

## 一句话总结

节点通过共同的“链选择标准”，在暂时分叉中逐渐选定同一份交易历史；PoW 带来的计算成本和随机性，使操纵这段历史变得困难且代价高昂。