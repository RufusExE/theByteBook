# 5.2 拜占庭将军问题

## 1. 什么是拜占庭将军问题

Lamport 认为“故事让问题变得欢迎”，因此他在提出观点和问题时常用故事背景吸引眼球（包括 Paxos 论文）所以，拜占庭将军问题，是 Lamport 在研究分布式系统容错性时编的一个故事。

:::tip 拜占庭将军问题描述

拜占庭帝国派出多支军队去围攻一个强大的敌人，每支军队有一个将军，但由于彼此距离较远，他们之间只能通过信使传递消息。敌方很强大，必须有超过半数的拜占庭军队一同参与进攻才可能击败敌人。在此期间，将军们彼此之间需要通过信使传递消息并协商一致后，在同一时间点发动进攻。
:::

回顾上面的问题，一群将军想要实现一个目标（一致进攻或者一致撤退），但是单独行动行不通，必须合作达成共识。由于存在叛徒，将军们不知道该如何达到一致。注意，这里的一致性才是拜占庭将军讨论的问题，如果叛徒的数量多到不可解决，那就是就是拜占庭（灭国）的问题了。那么我们的目标就是忠诚的将军们能够达成一致，对于那些忠诚们的将军来说，进攻或者撤退都可以，只要能够达成一致就没问题。

但仔细想想，只靠”**一致性**“就可以解决问题么？如果万事俱备，客观上每个忠诚的将军只要进攻了就一定能够胜利，但因叛徒的存在，它们都因”一致性“没有进攻；反之，条件不利，将军们不应该进攻，但却因叛徒的存在，所有人都”一致性“地进攻了。所以解决拜占庭将军们的问题，还要提出一个**正确性**的要求。


## 2. 二忠一叛变难题

为了更加深入的理解拜占庭将军问题, 我们把问题简化一下，假设有三个拜占庭将军，分别为 A、B、C。三个将军要决定的只有一件事情：明天是进攻还是撤退。为此将军们需要依据”**少数服从多数**“的原则投票表决，只要两个人意见达成一致就可以了。

举例来说，A、B 投票进攻，C 投票撤退：

- 那么 A 的信使传递给 B 和 C 的消息都是进攻。
- B 的信使传递给 A 和 C 的消息都是进攻。
- C 信使传递给 A 和 B 的消息都是撤退。 

<div  align="center">
	<img src="../assets/byzantine-1.svg" width = "400"  align=center />
</div>



按照少数服从多数原则，C 也会进攻，最终三位将军同时进攻，战争获得胜利。

可是，问题来了：假如三位将军中出现一位叛徒呢？叛徒的目标是破坏忠诚将军们之间的一致性达成，让拜占庭军队收到损失。


这个解决办法其实就是  Lamport 在论文中提到的口信消息型拜占庭问题之解：如果叛徒人数为 m，将军的人数小于或者等于 3m 时，叛徒便无法被发现，整个系统的一致性也无法解决。

看完故事回到现实，分布式系统领域中拜占庭将军问题中的角色与计算机世界的对应关系如下：

- 将军, 对应计算机节点。
- 忠诚的将军, 对应运行良好的计算机节点。
- 叛变的将军, 被非法控制的计算机节点。
- 信使被杀, 通信故障使得消息丢失。
- 信使被间谍替换, 通信被攻击, 攻击者篡改或伪造信息。


拜占庭将军问题描述的是最困难、最复杂的分布式故障场景，该场景除了存在故障行为，还存在恶意行为。存在恶意行为的场景中（比如数字货币、Web3等区块链技术中），我们必须使用拜占庭容错（Byzantine Fault Tolerance）算法。常用的拜占庭容错算法有 PBFT、PoW 算法。

在计算机分布式系统中，最常用的是非拜占庭容错算法，即故障容错（Crash Fault Tolerance，CFT）算法。CFT 算法解决的是分布式系统中存在故障，但不存在恶意节点下的分布式共识问题。也就是说这个场景可能会丢失消息或者消息重复，但不存在消息错误或者被伪造的问题。常见的 CFT 算法有 Paxos 算法、Raft算法、ZAB 协议等。

拜占庭的故事构造的如此成功，区块链开发者无人不知。Lamport 尝到了甜头，后来在《The Part-time Parliament》的论文中又讲了一个虚构的故事，而这也是我们下一节的内容。