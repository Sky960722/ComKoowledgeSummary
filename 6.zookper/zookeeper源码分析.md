# Zookeeper源码分析

## Paxos算法

- Paxos算法：一种基于消息传递且具有高度容错特性的一致性算法
- Paxos算法解决的问题：如何快速正确的在一个分布式系统中对某个数据值达成一致，并且保证不论发生任何异常，都不会破环整个系统的一致性

### 算法描述

- 在一个Paxos系统中，首先将所有节点划分为 Propser（提议者），Acceptor（接收者），和Learner（学习者）。（注意：每个节点都可以身兼数职）
- 一个完整的Paxos算法流程分为三个阶段：
- Prepare准备阶段
  - Porpser向多个Acceptor发出Propses请求Promise（承诺）
  - Acceptor针对收到的Propose请求进行Promise（承诺）
- Accept接受阶段
  - Proposer收到多数Acceptor承诺的Promise后，向Acceptor发出Propose请求
  - Acceptor针对收到的Propose请求进行Accept处理
- Learn学习阶段：Proposer将形成的决议发送给所有Learners

### 算法流程

1. Prepare Proposer生成全局唯一且递增的Proposal ID，向所有Acceptor发送Propose请求，这里无需携带提案内容。只携带Proposal ID即可
2. Promise：Acceptor收到Propose请求后，做出"两个承诺，一个应答"
