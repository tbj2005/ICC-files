GPT3：1750亿参数

PaLM：5400亿参数

LLM训练三个步骤：预训练（最费时）、微调、服务

预训练一个 PaLM 模型需要$2.5\times 10^{24}$次浮点数运算，在 6144 张 TPU 上训练了 64 天。

微调以提升模型在真实世界的表现。

部署成网页或API服务，为给定的输入查询提供推理结果。

A100扩展性和性能都最佳，7BLlama2 模型使用8张卡数据并行总共能提供约$16\times10^3$tokens/s的速度，不使用优化技术。

单 GPU 速率：约2k token/s

350序列长度，64batchsize，分8卡计算，8卡总处理速率最高取640000 tokens/s，此时迭代一轮所需计算时间在十毫秒级，考虑到卡的性能和优化技术的使用，迭代一轮的时间应该在十毫秒到百毫秒之间。

重构所需时间在百毫秒级。

一次迭代通信时间是十毫秒到百毫秒间。

在 MoE 中, 专家并行的通信量是远小于数据并行的。
