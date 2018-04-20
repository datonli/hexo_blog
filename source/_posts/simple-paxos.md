---
title: Simple Paxos
date: 2018-04-20 23:09:01
tags: distributed
---

搞分布式系统的，一定会听过Paxos——非常有名的一致性协议（consesue protocol），根据Lamport的paper——Paxos Made Simple，总结一下Paxos的理论。后面有计划再写一篇有关Paxos实现的博客，敬请期待。

### Paxos解决什么问题
---
paxos解决的是数据一致性和系统可用性这两个问题。

假设一个系统，简单的数据库服务器来说，单节点接收client的写数据，为了提升速度需要接收一段时间数据缓冲在内存中，然后在写到磁盘中进行持久化。

这时候因为服务器随时会down，内存中的数据随时会丢，系统可用性差；

好，这时一个很顺其自然的思路就是：主备服务器。一台不行我上2台，3台，N台，主挂了之后备机上。这时系统可用性提升了，但是有带来另外一个问题——数据一致性。

N台服务器，主机接收写数据，这些数据需要同步到N-1台备机上（不同步一旦主机down了可用性还是差），如何同步才能保证主机在随时会down的情况下，切到备机而数据不丢？

---
### 一致性和可用性方案的演进
---
1. 主机把数据送到第一台备机，保证数据送到，然后就不管了：1->2，2->3->...->N，hdfs才用的是这种方案；并非不可用，只是如果发生连续down服务器几台还是要跪，另外还有同步数据耗时很长；
2. 主机把数据同时送给N-1台备机，等待备机全回复ack，主机再去处理下一个写请求；这种方案保证了数据强一致性，但主机的可服务时间很少（因为如果在送数据等ack的时候接收写数据，这些数据又不能post到备机，万一挂了，还是影响可用性），系统可用性也很低；
3. 在2的基础上，主机还是把数据同时送给N-1台备机，只等待N/2台机器返回ack，就可以继续接收写数据请求；这种时候系统可用性还是不高，但是已经do my best了；这时也有问题，没有返回ack的剩下的N/2-1台机器怎么办？如何从其他已经ack的机器拿到最新数据，保证数据一致性？（raft算法）
4. 在3的基础上，因为1到3都是基于只有一个主机提供服务，而备机不能提供服务的，what if N台机器都可以接收写请求，接收到写请求之后分别尝试去post data到其他机器？另外和3一样的，没有ack的机器如何从其他机器learn newest data保证数据一致性？（pexos算法）
 
以上的几个方案就是就是容易想到的解决问题的思路了，可以看到raft和pexos就出来了。只是完成这个过程并不容易，看看Lamport老头子1998年写的pexos算法没人看懂、发paper都发不出去就知道了。

---
### Pexos算法
---
先来明确几个概念：

```
agent：代表一台机器
proposer：向其他机器发起同步数据请求，等待ack；
accepter：接收其他机器发送的同步数据请求，ok的话返回ack；
proposal：被同步的请求数据；
learner：没来得及ack或者失败之后，需要向别的机器learn newest data的机器
```

还要明确一下前提：
```
paxos算法只适用于非拜占庭模型（non-Byzantine model）：
1. 机器（agent）可以有各种不可用（运行可快可慢、随时down和restart）；
2. 数据（message）可以有各种问题（数据发送耗时任意长、可能重复、可能丢失），但是数据不会损坏（包括假数据）；
3. 各种failure都有可能，但是要确保：
    1）机器是等幂的（机器不欺骗，不能对A返回3，对B返回4）
    2）拿到了数据就一定是可信的（信道可靠）
```

进入真正的算法环节。

---
### Choosing a value
---
如何在众多机器之间，确定一个value？

1. 最简单方法，只有一个acceptor，并且把接收到的第一个proposal作为chosen value；很简单，但是也没法满足可用性（acceptor挂了）。
2. 多个acceptor，一个proposer发送proposal到多个acceptor，等large enough的accepters接收这个value，就ok了；但是how large is enough？为了确保只有一个value被chosen，需要large到majority的agents，因为假如存在两个majority accept了两个不同的value，那么一定有一部分agents accept了两个value；这时就有一个accept value顺序的问题，引入一个原则：
```
P1. An acceptor must accept the first proposal that it receives.
```
P1又带来了一个问题，多个值同时被多个proposer proposed，导致每个acceptor都accept了自己收到的第一个proposal，但是却没有一个value被majority agents accept的？即使是两个proposal都分别被half的agents accept了（2N+1个agents，刚好有一个down了），刚down掉又拉起来的agent也不知道该以哪个value为准（to learn）？
3. 对于2中提到的问题，为了避免这种情况，给proposal增加一个属性——proposal number，这个number是从proposer记录的状态来的，让proposers记住现在自己的状态是处于哪个proposal number了，那么下一次提交proposal number时+1，送到acceptors；**从技术实现上保证不同的proposal一定有不同的proposal number**。
```
保证proposal number一定不同，技术实现的考虑：
每个proposer持有不同的proposal number增长步长。
```
至此，我们可以保证，一个被majority agents accpet的proposal一定可以作为chosen value了（即使是在2中提到的极端情况，两个proposal被accpet by 2个majority agents，也可以根据proposal number来选择只取一个或者两个都保留）。

这里proposal number是一个关键，在介绍这个之前，先明确一点，所有chosen proposals一定有same values：
```
P2. If a proposal with value v is chosen, then every higher-numbered proposal that is chosen has value v.
```
由于proposal number保证了严格有序，chosen proposals在每个agent上都应该是一致的（最终一致性）。由于proposal需要acceptor才能被accept，以此可以推出：
```
P2a. If a proposal with value v is chosen, then every higher-numbered proposal accepted by any acceptor has value v.
```
此时，保证了所有的acceptor在accept新的proposal之前，需要先保证旧的chosen proposals要已经同步好了。

`有个proposal number的小问题，一台agent如果没有update到最新的data，是否可以作为proposer发送proposal？`

假如，有一个新的proposer（挂了之后重新开始）发了一个higher number proposal with different value，对于一个没更新到最新数据的acceptor c来说，P1要求c接收这个value，但是这违反了P2a。这时需要有更严格的原则加强P2a：
```
P2b. If a proposal with value v is chosen, then every higher-numbered proposal issued by any proposer has value v.
```
发布higher number proposal的proposer必须先把前面的chosen value同步好先。

所以，依据上面的P2、P2a、P2b三个原则，算法要求**作为proposer和acceptor都必须先把chosen values先补齐（learn）先**（Lamport老爷子用了一堆prove和P2c来证明这个事情）。

对于acceptor来说，无法预测该chosen哪个值，只有proposer接到了majority回复之后才知道；要保证acceptor不会反反复复，proposer要求acceptor不能再接收比刚发送的proposal number更小的proposal了，以proposal number保证旧的且未被接收的proposal不会再被接收。

有这个基础，可以说明proposer的算法逻辑了：
```
1. 当proposer发送一个proposal number n的proposal，要求acceptor回复：
    （a）acceptor保证不再接收proposal number < n的proposal；且
    （b）如果有小于n的最大的proposal已经被accept了，携带上这个proposal；
    这个就是proposer发送给acceptor的prepare request（携带n proposal），和返回的回复。
2. 当proposer接收到majoriyt的acceptors回复之后，proposer再发布（issue）一个proposal（带着number n和value v，其中v是response返回最大proposal number）到一部分acceptors（和返回reponse的acceptors是同一堆）中，这个请求叫accept request。
```
再看看acceptor的算法逻辑：
```
由proposer算法逻辑可知，acceptor可以接受两种request：prepare request和accept request。
acceptor接收prepare request，看看是否比现在promise的proposal number要大，是的话就accept并返回response；不是的话直接忽略。

P1a. An acceptor can accept a proposal numbered n iff it has not responded to a prepare request having a number greater than n.

```

**小结：choosing a value分两个阶段**
```

Phase 1. 
    (a) A proposer selects a proposal number n and sends a prepare request with number n to a majority of acceptors.
    (b) If an acceptor receives a prepare request with number n greater than that of any prepare request to which it has already responded, then it responds to the request with a promise not to accept any more proposals numbered less than n and with the highest-numbered proposal (if any) that it has accepted.

Phase 2. 
    (a) If the proposer receives a response to its prepare requests (numbered n) from a majority of acceptors, then it sends an accept request to each of those acceptors for a proposal numbered n with a value v, where v is the value of the highest-numbered proposal among the responses, or is any value if the responses reported no proposals.
    (b) If an acceptor receives an accept request for a proposal numbered n, it accepts the proposal unless it has already responded to a prepare request having a number greater than n.
```

看看proposer和acceptor的数据结构：
1. 对于acceptor来说，需要记住：the highest- numbered proposal that it has ever accepted and the number of the highest- numbered prepare request to which it has responded
2. 对于proposer来说，只需要记住下一次发起proposal的number就可以了（保证不重复发），不记录其他信息，即便是在protocol进行中，也随时可以aboden。
3. 因为第2点，所以当一个acceptor接收到一个higher numbered proposal之后，可以通知proposer放弃掉之前的proposal，另外再重新发起？

`小问题：value v有什么作用？`

---
#### Learning a Chosen Value
---
为了去learn那些已经chosen的values，需要先找出被majority acceptors accepted的proposal。
1. 最obvious的方法是：让每个acceptor一旦accept proposal & choose a value就通知所有learners，这种方案可以快速通知learner和更新数据，但是这需要大量的开销（acceptor_num * learner_num）；
2. 另一种方法是：所有acceptors关联到一个learner上，一旦有了新的chosen proposal就发给learner，让这个learner通知其他的learners，这样做降低了可靠性，但是开销大量降低（acceptor_num + learner_num）；
3. 更通用的方法是：所有acceptors关联到一个learner set，每次有chosen proposal通知到这个learner set，learner set每一个learner都可以通知其他的learners；这样在保证一定的可靠性前提下降低了开销（acceptor_num * learner_set_num + otherwise_learner_num）。

---
#### Progress
---
工程实践上，需要考虑一些问题。比如：2个proposers，分别是p和q，对于acceptors，p先发布n1，完成了阶段1，正准备发accept request，这时q就发了n2 (n2 > n1)，acceptors收到n2之后，p发过来的accept request就会被acceptors忽略；这时p再重新发一个n3 (n3 > n2)，发到acceptors导致q发的accept request也被忽略，周而复始，结果只是proposal number在不停增加，没有一个proposal被chosen的。

为了防止出现这种情况，需要找出一个proposer专门发布proposals的。

---
#### The Implementation
---
在Paxos consensus algorithm中，每个进程扮演着proposer/acceptor/learner中的一个的角色，算法选出一个leader（pexos用paxos来选leader），同时扮演着特别的proposer和learner的角色。

在acceptor上还需要有一个stable storage来保存一些必须记住的infos，这些信息会在落盘之后才回复reponse。

为了让proposal number不重复，会让proposer选择一个离散的数据集，相互间不会重复，这样proposer只需要在stable storage中保存highest-numbered proposal，这样开始下一个proposal用更大的number就行。

---
参考文献：[Paxos Made Simple - Leslie Lamport](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf)