# SERO Staking 及 Staking Service 机制简要说明

SERO 主网正式发布版本和之前Beta版本核心区别在于共识机制的升级，加入 `Staking` 机制通过对 PoW 的出块进行验证的方式参与共识，为底层网络和生态角色之间互相协作提供更好的一种激励平衡关系，并能在生态发展的过程中不断动态优化调节。



## Original Purpose

在现阶段需要更多的机制来确保网络的安全以及应对一些其它问题，这些问题会包括：

* **算力垄断：**目前 POW 挖矿的整个生态已经成为一个相对不平衡的领域，矿工激励本身是一种非常公平的机制，但由于现阶段区块链生态价值缺乏合理的评价体系，因此单纯矿工激励的机制使网络安全存在相当的隐患，譬如 51% 攻击的概率提升，我们并不认为没有充分的利益驱动所以 51% 攻击的发生概率降低是一种合理的解释。

* **能源支出巨大：**挖矿是一个计算密集型的产业，需要消耗大量的资源。以 BTC 挖矿为例，年耗电量约为 49 TWh（万亿瓦时），几乎与新加坡一年的耗电量相当，因此部分使用同样可以量化成本的 PoS 机制可以有效降低实际能源消耗，用一个可能不完全恰当的比喻，就好比我们可以使用火力发电的同时，也可以使用更清洁的水力或者风力发电，我们必须理解的是 PoS 的参与并非是没有成本的，这如同一次性投入固定资产收益和持续通过劳动产生收益更像是一个完整的经济体。

* **奖励的公平性：**在之前发布的 `BetaNet-R7.1` 中，我们已经将出块奖励调整为和全网难度相关，以保障 SERO 币的产量在经济流通市场保持合理的（和生态发展相关）通货膨胀水平，在新加入的 `Staking` 机制中，我们同样将 `Staking` 的区块奖励定义为一个自动和全网 PoS 参与占比形成一种自动平衡的关系，本质上这将进一步有效对通胀水平进行了调节，如果说算力（生产力）大小和市场需求保持了一种正相关，对固定资产投资占比则影响了市场的流动性，在市场流动性差的时候我们应该降低对固定资产投资的收益率。

* **网络安全：**虽然一直认为作恶动机是和成本及利益回报相关，但实际对于一个旨在长远发展的公链来说，我们必须确保网络总是能够处于尽可能的安全状态，而不是仅仅确保威胁网络安全的成本高于某个流通市值的比例。

  记账权掌握在矿工手中，51% 攻击者存在远低于流通市值成本攻击的机会——通过短时间内操控算力进行双花，只需要很小的一笔算力租借费用，可以带来异常高的“投资回报率”。而加入 `Staking` 机制，可以使短期算力形成 51% 攻击成本进一步大幅提升，从而使发生攻击的成功率接近于零。



## SERO Staking
SERO 的 `Staking` 运作方式简单来说，主要是由 POW 矿工负责出块，但是块的合法性须由 PoS 参与者确认，由于 PoS 矿工都是 SERO 的持有者，而在出块过程中，验证者的选取过程有很大的随机性，因此攻击成本几乎已经和短期算力关系不大，如果要确保自己伪造的出块有效，则同时需要获得足额 PoS 参与占比。

具体的实现过程原理是，用户需要锁定自己的一部分 SERO 币竞拍区块选票 `ValidateShare`，PoW 矿工每生成1个区块时，需要从票池随机选取 3 张 `ValidateShare`，对区块进行一次有效验证的投票，投票完成后，用户竞拍 `ValidateShare` 的 SERO 代币会得到归还，并获得相应的区块 PoS 奖励，值得一提的是，为了有效降低全网交易数，以及使验证的有效性在网络区块进一步固化后被确认，PoS 购买选票和区块奖励的 SERO 币，将会在用户参与 PoS 后大约每一周自动结算一次并归还到账户。

以上这部分 PoS 区块奖励占挖矿产出 Token 2/8 占比，如果一个区块奖励为 72 个币的话，PoW 矿工将会获得其中的 45 个，PoS 参与者将会获得其中的 18 个，其余 9 个将会奖励 `StakingPool`（详见后文论述）。

票池并没有固定的大小，大致可以推算出，平均每个 `ValidateShare` 将会锁定那部分 SERO 代币 30 天的时间，随着全网流通的释放和市场波动影响，`ValidateShare` 的购买价格会和一段时间内集中购买的数量相关，在购买集中的时段，每张 `ValidateShare` 的购买价格会有所提升，从而可以实现有效且自动调节 SERO 代币的全网流通水平。

在 `ValidateShare` 被选中对区块进行投票时，`ValidateShare` 的持有人需要保持全节点在线并由账户自动完成有效投票，如当时无法保持在线投票，则会被作为弃票处理，并无法获得区块奖励。

因此另一个选择是，在发起 PoS 参与（购买 `ValidateShare` 时），选择权益池节点为自己履行投票工作，这样自己就无须保持全节点在线也能获得 PoS 收益。



## SERO StakingNode
在 `ValidateShare` 被选中需要进行投票的时候，用户需要通过节点在线以参与投票，如果无法投票，则用户不能获得奖励，由于考虑到节点在线的成本，以及需要及时参与投票验证区块所需要的网络和硬件条件，用户可以选择由特殊的节点代为投票，这类节点我们称之为 `StakingNode`。

![图片](https://uploader.shimo.im/f/2JtfYzaqhBkAQwoK.png!thumbnail)
和现有的大部分 `STAKING` 共识的区块链网络不同，SERO 的 `STAKING` 投票节点，也就是`StakingNode`，可以由任意节点主动在网络上通过注册的方式被选取，注册 `StakingNode` 时，网络会根据当前全网 `StakingNode` 数量和 POS 参与总量自动计算注册时需要抵押的 SERO 代币，并向注册者返回注册结果状态。

成功注册为 `StakingNode` 后，可以向普通节点 `Node` 提供自己的节点地址，在普通节点参与 POS时，需要在提交购买选票时，需要填写（选择）一个为其代为投票的有效 `StakingNode` 地址（默认为自己的地址），以通过 `StakingNode` 的方式参与 PoS ，以获得更大的投票成功率。 

全网所有的 `StakingNode` ，将会分享区块奖励中，对于 `StakingNode` 这类节点由于其提供了稳定的投票通道并公正分配 POS 奖励的服务价值，获得相应的区块奖励，这部分奖励约为所有区块奖励中的 1/8（暂定）部分。

具体对 `StakingNode` 的奖励规则为，每个出块奖励分配中，5/8 将分给提供 POW 算力获得记账权的矿工，2/8 将分给随机选中为该区块投票的3张选票的主人，1/8 将分给为这 3 张选票提供投票服务的 StakingNode。`（注：由非StakingNode进行投票的节点将无法获得这部分1/8的奖励，由此产生的多余的1/8将被销毁）`

因此每个 `StakingNode` 所获得的奖励，大致概率上将由其在整个票池中，选择该 `StakingNode` 总票数的占比决定。

![图片](https://uploader.shimo.im/f/JvDct2C53EIPp6hu.png!thumbnail)

## 如何注册 StakingNode

`SERO Staking` 机制是去中心化实现的，因此 `StakingNode` 的注册方式也并不需要向任何一个中心化机构申请。

**具体注册方式为：**需要由全节点钱包发起（或权益池软件），在网络上质押一段时间（大约为 30 天 ）一定数量的 SERO ，自动注册成为一个权益池账户，就可以自动获得一个 `StakingNode` 的账户节点身份，具体的支付数量由全网动态计算并提示注册者。

权益池注册所需要质押的 SERO 为 20 万枚，满足以下条件后，这部分币可以返回账户：
1. 由权益池账户发起关闭权益池；
2. 所有指定该权益池履行投票义务的 `ValidShare` 完成投票清算；
3. 从权益池注册开始起已达到 30 天；

获得 `StakingNode` 身份后，在普通用户参与 POS 进行买票的时候，就可以设置 `StakingNode` 地址，由 `StakingNode` 为选票进行代理投票。

所以为了确保 `StakingNode` 能够稳定提供代理投票服务，以确保 POS 参与者的有效投票能够获得收益，`StakingNode` 需要保持节点长期在线，如果投票率低于某个数值，将会触发惩罚机制。