最近看到claude把数学千禧年问题的黎曼猜想的零点个数比例从41.4%推进到了67.2%。作为一个nlp算法工程师，我觉得这里面的一些事情值得琢磨

- 为什么claude能超过人类数学家，把这个问题往前推进这么多：训练数据、算法哪个更重要呢

- 如果rl相关的算法更重要一些，那么claude是否用到了超过rlvr的一些算法，根据论文（does reinforcement learning really incentives ...）提到的，当k逐渐增大，base模型的pass@k最终会超过instruct或者think模型的pass@k，以及rl里的一些算法，这意味着rlvr算法会导致模型产生路径依赖，降低生成解的多样性。既然如此，那么要么claude训练的时候使用的是一种超越rlvr的rl算法，要么是其他因素导致的claude超越人类数学家。

- 如果pretrain数据更重要一些，那么世界上的所有知识是不是就只需要通过一些公理（或者原子性的能力）获得，现有的一些llm使用几十T的训练token实际上是徒劳且低效的。因为在准备pretrain语料的时候不可能把黎曼猜想的点>=67.2%放进训练数据，因为人类数学家还未验证。

- 那么还有一种可能：agent架构的设计问题。我的想法是增加agent训练时间，相当于把pass@k的k增加了，变成pass@budget，claude并行跑多个可能的解题路径，然后通过验证器验证并汇总。


- 对我的启发
1. 数据评估是不是可以更新？通过agent执行里的原子能力（例如分解问题、失败重试、in context learning）来评估而不（只）是某个benchmark的具体分数，目前也有比较多的评估agent系统的benchmark。
