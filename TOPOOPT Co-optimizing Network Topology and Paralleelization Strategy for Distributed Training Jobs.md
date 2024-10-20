### TOPOOPT: Co-optimizing Network Topology and Parallelization Strategy for Distributed Training Jobs

TOPOOPT 是我们提出的一种全新的用于深度神经网络训练负载的直连网络，它通过三个维度（计算、通信、网络拓扑）联合优化了分布式训练过程。

目前训练负载不能满足在胖树互联的设计中产生的数据中心流量：1. 大型 DNN 训练的通信负载是随着 worker 数目的增加动态增长的；2.  DNN 训练的通信模式是由并行策略决定的。TOPOOPT 使用可重构光交换机和配线架为每个训练业务创造专用的分区，并在每个分区联合优化了拓扑和并行策略。为了达成这一目标，我们尽力解决了寻找最佳拓扑的算法挑战，例如怎样流量计算、通信和拓扑维度的巨大搜索空间，还有多种操作挑战，例如光交换技术怎样和 DNN 模型的通信模式匹配。

我们将拓扑和并行策略共同优化问题描述出一个离线交替优化架构。我们的优化策略在优化并行策略和优化网络拓扑间选取。先在假设固定拓扑的情况下，搜索并行策略空间，然后将通信需求反馈到寻拓扑算法中。更新后的拓扑然后输回并行策略搜索算法。这个交替过程会重复，直到系统收敛到一个已经优化好的并行策略和拓扑。

寻找和优化 DNN 网络拓扑是困难的，因为理想的网络拓扑需要同时满足两种条件：1. 能高效完成大规模 AllReduce 传输；2. 确保在模型并行传输中有较小的跳数。为了满足这些目标，我们提出了一种新的基于群论原理的技术，叫做 TotientPerms，可以利用 AllReduce 通信的可变性。我们的 TotientPerms 方案建立了一系列 AllReduce 排列，它们不仅可以高效地传输 AllReduce 的流量，还能很好的安放以保证模型并行流量的传输。

光交换机只支持点对点通信，为了支持参与作业的所有主机之间的任意通信，我们让 TOPOOPT 的主机充当中继站，转发不属于它们的流量。基于主机的转发为 RDMA 流量带来了新的挑战，因为 RDMA 网卡会丢弃不属于自己的数据包。为了实现基于主机的 RDMA 转发，我们利用了现代网卡的网络分区 (NPAR) 功能，为 RDMA 数据包转发创建了一个高效的逻辑叠加网络。

迭代过程存在两种依赖：1. 在前向和反向传播步骤中计算激活值和梯度，每个输入样本都需要这种数据依赖性；2. 通过AllReduce步骤在一批样本处理后同步模型权重在加速器之间的数据。根据并行化策略的不同，这些数据依赖可能导致本地内存访问或者加速器之间的通信。

DLRM 流量模式：DLRM 是一种个性化和推荐模型，一般有千亿级参数，这是因为它们巨大的嵌入表。只是用数据并行来做分布式会导致大规模的 AllReduce 传输。

大多数业务的 worker 使用在 32 到700 间（Meta 集群），符合近期工业主流宣布的数值，每个 worker 上只有单个 GPU 。大多数业务都需要十小时以上才能完成。随着 GPU 数目的提高，网络很快就占据了训练迭代时间的很大一部分。

在 DNN 训练工作负载中，每个服务器的度数通常小于训练期间与服务器通信的邻居总数。为了保证无阻塞，需要使用基于主机的转发技术可以为两个服务器提供非直连服务
