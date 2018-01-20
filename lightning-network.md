# 比特币闪电网络

闪电网络是一种端到端连接的双向支付通道的可路由网络。这样的网络可以允许任何参与者穿过一个通道路由到另一个通道进行支付，而不需要信任任何中间人。闪电网络由Joseph Poon和Thadeus Dryja于2015年2月首次描述，其基础是许多其他人提出和阐述的支付通道概念。

“闪电网络”是指路由支付通道网络的具体设计，现已由至少五个不同的开源团队实施。这些的独立实施是由“闪电技术基础”（BOLT）论文中描述的一组互通性标准进行协作。

闪电网络的原型实施已经由几个团队发布。现在，这些实现只能在testnet上运行，因为它们使用segwit，还没有在比特币区块主链（mainnet）上激活。

闪电网络是实现可路由支付通道的一种可能方式。还有其他几种旨在实现类似目标的设计，如Teechan和Tumblebit。

### 12.7.1 闪电网络示例

让我们看看它是如何工作的。

在这个例子中，我们有五个参与者：Alice, Bob, Carol, Diana, and Eric。这五名参与者已经彼此之间开设了支付通道。Alice和Bob有支付通道。Bob连接Carol，Carol连接到Diana，Diana连接Eric。为了简单起见，我们假设每个通道每个参与者都注资2个比特币资金，每个通道的总容量为4个比特币。

下图12-9显示一系列通过双向支付的通道连接在一起形成闪电网络以支持一笔从Alice到Eric的付款 展示了闪电网络中五名参与者，通过双向支付通道连接，可从Alice付款到Eric（路由支付通道（闪电网络））。

[![图12-9显示一系列通过双向支付的通道连接在一起形成闪电网络以支持一笔从Alice到Eric的付款](https://camo.githubusercontent.com/1f2a5c1dbf6f8c03097444f78075ff7321f7dc7c/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313738353935392d656466393061356365666635613336652e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/1f2a5c1dbf6f8c03097444f78075ff7321f7dc7c/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313738353935392d656466393061356365666635613336652e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

Alice想要支付给Eric1个比特币。 不过，Alice并未通过支付通道连接到Eric。 创建支付通道需要资金交易，而这笔交易必须首先提交给比特币区块链。 Alice不想打开一个新的支付通道并支出更多的手续费。 有没有办法间接支付Eric？

下图12-10 显示了通过在连接各方参与者的支付通道上通过一系列HTLC承诺将付款从Alice路由到Eric的逐步过程。

[![图12-10 ](https://camo.githubusercontent.com/6c36ebad622e0fcbabea4123ce1339bb0b1a3eb0/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313738353935392d346137363030663431656538343939612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/6c36ebad622e0fcbabea4123ce1339bb0b1a3eb0/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313738353935392d346137363030663431656538343939612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

Alice正在运行闪电网络（LN）节点，该节点正在跟踪其向Bob的付费通道，并且能够发现支付通道之间的路由。Alice的LN节点还具有通过互联网连接到Eric的LN节点的能力。 Eric的LN节点使用随机数生成器创建一个秘密R。Eric的节点没有向任何人泄漏这个秘密。相反，Eric的节点计算秘密R对应的哈希H，并将此哈希发送到Alice的节点（请参阅图12-10步骤1）。

现在Alice的LN节点构建了Alice的LN节点和Eric的LN节点之间的路由。所使用的路由算法将在后面进行更详细的解释，但现在我们假设Alice节点可以找到一个高效的路由。

然后，Alice的节点构造一个HTLC，支付到哈希H，具有10个区块时间的退款超时（当前块+10），数量为1.003比特币（参见图12-10的步骤2）。额外的0.003将用于补偿参与此支付路由的中间节点。Alice将此HTLC提供给Bob，从和Bob之间的通道余额中扣除1.003比特币，并将其提交给HTLC。 该HTLC具有以下含义：“如果Bob知道秘密，Alice将其通道余额的1.003支付给Bob，或者如果超过10个区块时间后，则退还入Alice的余额”。 Alice和Bob之间的通道余额现在由承诺交易表示，其中有三个输出：Bob的2比特币余额，Alice的0.997比特币余额，Alice的HTLC中承诺的1.003比特币。承诺在HTLC中的金额从Alice的余额中被减去。

Bob现在有一个承诺，如果他能够在接下来的10个区块生产时间内获得秘密R，他可以获取Alice锁定的1.003。手上有了这一承诺，Bob的节点在和Carol的支付通道上构建了一个HTLC。Bob的HTLC提交1.002比特币到哈希H共9个区块时间，这个HTLC中如果Carol有秘密R她可以兑换（参见图12-10步骤3）。Bob知道，如果Carol要获取他的HTLC，她必须出示秘密R。如果Bob在9个区块的时间内有R，他可以用它来获取Alice的HTLC给自己。通过承诺自己的通道余额9个区块的时间，他也赚了0.001比特币。如果Carol无法获取他的HTLC，并且他也无法获取Alice的HTLC，那么一切都将恢复到以前的通道余额，没有人会亏损。 Bob和Carol之间的通道余额现在是：2比特币给Carol，0.998给Bob，1.002由Bob承诺给HTLC。

Carol现在有一个承诺，如果她在接下来的9个区块时间内获得R，她可以获取Bob的锁定1.002比特币。现在她可以在她与Diana的通道上构建HTLC承诺。她提交了一个1.001比特币的HTLC到哈希H，共计8个区块时间，如果Diana有秘密R ，她就可以兑换（参见图12-10步骤4）。从Carol的角度来看，如果能够实现，她就可以获得的0.001比特币，否则也没有失去任何东西。她提交给Diana的HTLC，只有在R被泄漏的情况下才可行，到那时候她可以从Bob那里索取HTLC。Carol和Diana之间的通道余额现在是：2给Diana，0.999给Carol，1.001由Carol承诺给HTLC。

最后，Diana可以提供给Eric一个HTLC，承诺1比特币，7个区块时间，到哈希H（参见图12-10的步骤5）。Diana与Eric之间的通道余额现在是：2给Eric，1给Diana，1由Diana承诺给HTLC。

然而，在这条路上，Eric拥有秘密R，他可以获取Diana提供的HTLC。他将R发送给Diana，并获取1个比特币，添加到他的通道余额中（参见图12-10的步骤6）。通道平衡现在是：1给Diana，3给Eric。

现在，Diana有秘密R，因此，她现在可以获取来自Carol的HTLC。Diana将R发送给Carol，并将1.001比特币添加到其通道余额中（参见图12-10的步骤7。现在Carol与Diana之间的通道余额是：0.999给Carol，3.001给Diana。Diana已经“赚了”参与这个付款路线0.001比特币。

通过路由回传，秘密R允许每个参与者获取未完成的HTLC。Carol从Bob那里获取1.002个比特币，将他们通道余额设为：0.998给Bob，3.002给Carol（参见闪电网络步骤8）。最后，Bob获取来自Alice的HTLC（参见闪电网络步骤9）。他们的通道余额更新为：0.997给Alice，3.003给Bob。

在没有向Eric打开通道的情况下，Alice已经支付了Eric 1比特币。付款路线中的中间方不必要互相信任。在他们的通道内做一个短时间的资金承诺，他们可以赚取一小笔费用，唯一的风险是，如果通道关闭或路由付款失败，退款有段短短的延迟时间。

### 12.7.2 闪电网络传输和路由

LN节点之间的所有通信都是点对点加密的。 另外，节点有一个长期公钥，[它们用作标识符并且彼此认证对方](http://bit.ly/2r5TACm)。

每当节点希望向另一个节点发送支付时，它必须首先通过连接具有足够容量的支付通道来构建通过网络的路径。节点宣传路由信息，包括他们已经打开了什么通道，每个通道拥有多少容量，以及他们收取多少路由支付费用。路由信息可以以各种方式共享，并且随着闪电网络技术的进步，不同的路由协议可能会出现。一些闪电网络实施使用IRC协议作为节点宣布路由信息的一种方便的机制。路由发现的另一种实现方式是使用P2P模型，其中节点将通道宣传传播给他们的对等体，在“洪水泛滥”模型中，这类似于比特币传播交易的方法。未来的计划包括一个名为[Flare](http://bit.ly/2r5TACm)的建议，它是一种具有本地节点“邻居”和较长距离的信标节点的混合路由模型。

在我们前面的例子中，Alice的节点使用这些路由发现机制之一来查找将她的节点连接到Eric的节点的一个或多个路径。一旦Alice的节点构建了路径，她将通过网络初始化该路径，传播一系列加密和嵌套的指令来连接每个相邻的支付通道。

重要的是，这个路径只有Alice的节点才知道。付款路线上的所有其他参与者只能看到相邻的节点。从Carol的角度来看，这看起来像是从Bob到Diana的付款。 Carol不知道Bob实际上是中继转发Alice的汇款。她也不知道Diana将会向Eric中继转发付款。

这是闪电网络的一个重要特征，因为它确保了付款的隐私，并且使得很难应用监视，审查或黑名单。但是，Alice如何建立这种付款途径，而不向中间节点透露任何内容？

闪电网络实现了一种基于称为Sphinx的方案的洋葱路由协议。该路由协议确保支付发送者可以通过闪电网络构建和通信路径，使得：

- 中间节点可以验证和解密其部分路由信息，并找到下一跳。
- 除了上一跳和下一跳，他们不能了解作为路径一部分的任何其他节点。
- 他们无法识别支付路径的长度，或者他们自己在该路径中的位置。
- 路径的每个部分被加密，使得网络级攻击者不能将来自路径的不同部分的数据包彼此关联。
- 不同于Tor（互联网上的洋葱路由匿名协议），没有可以被监视的“退出节点”。付款不需要传输到比特币区块链，节点只是更新通道余额。

使用这种洋葱路由协议，Alice将路径的每个元素包裹在一层加密中，从尾端开始倒过来运算。她用Eric的公钥加密了Eric的消息。该消息包裹在加密到Diana的消息中，将Eric标识为下一个收件人。给Diana的消息包裹在加密到Carol的公钥的消息中，并将Diana识别为下一个收件人。对Carol的消息被Bob的密钥加密。这样一来，Alice已经构建了这个加密的多层“洋葱”的消息。她发送给Bob，他只能解密和解开外层。在里面，Bob发现一封给Carol的消息，他可以转发给Carol，但不能自己破译。按照路径，消息被转发，解密，转发等，一路到Eric那里。每个参与者只知道各自这一跳的前一个和下一个节点。

路径的每个元素包含承载于HTLC的必须扩展到下一跳的信息，HTLC中的要发送的数量，要包括的费用以及CLTV锁定到期时间（以块为单位）。随着路由信息的传播，节点将HTLC承诺转发到下一跳。

在这一点上，您可能会想知道节点怎么知道路径的长度及其在该路径中的位置。毕竟，他们收到一个消息，并将其转发到下一跳。难道它不会将路径缩短，或者允许他们推断出路径大小和位置？为了防止这种情况，路径总是固定在20跳，并用随机数据填充。每个节点都会看到下一跳和一个要转发的固定长度的加密消息。只有最终的收件人看得到没有下一跳。对于其他人来说，似乎总是有20多跳要走。

### 12.7.3 闪电网络优势

闪电网络是第二层路由技术。它可以应用于支持一些基本功能的任何区块链，如多重签名交易，时间锁定和基本的智能合约。

如果闪电网络搭建在在比特币网络之上，则比特币网络可以大大提高容量，隐私性，粒度和速度，而不会牺牲无中介机构的无信任操作原则：

**隐私** 闪电网络付款比比特币区块链的付款更私密，因为它们不是公开的。虽然路由中的参与者可以看到在其通道上传播的付款，但他们并不知道发件人或收件人。

**流动性** 闪电网络使得在比特币上应用监视和黑名单变得更加困难，从而增加了货币的流动性。

**速度** 使用闪电网络的比特币交易将以毫秒为单位，而不是分钟，因为HTLC在不用提交交易到区块上的情况下被结算。

**粒度** 闪电网络可以使支付至少与比特币“灰尘”限制一样小，甚至更小。一些建议允许子聪级增量（subsatoshi increments）。

**容量** 闪电网络将比特币系统的容量提高了几个数量级。每秒可以通过闪电网络路由的付费数量没有具体上限，因为它仅取决于每个节点的容量和速度。

**无信任操作**闪电网络在不需要互相信任就可以作为对等体使用的节点之间使用比特币交易。因此，闪电网络保留了比特币系统的原理，同时显著扩大了其操作参数。

当然，如前所述，闪电网络协议不是实现路由支付通道的唯一方法。其他被提出的系统包括Tumblebit和Teechan。然而，在这个时候，闪电网络已经部署在testnet上了。几个不同的团队已经开发了正在竞争的LN实现，并且正在努力实现一个通用的互操作性标准（称为BOLT）。闪电网络很可能是第一个部署在生产实际中的路由支付通道网络。