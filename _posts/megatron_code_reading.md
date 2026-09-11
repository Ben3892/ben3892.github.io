# megatron代码阅读
按 **NVIDIA 的 Megatron-LM / Megatron Core** 来讲，路径以当前官方 `main` 分支为参考。建议实际阅读时固定一个版本，方便对照源码和做笔记。

**阅读主线是：先追踪一次训练迭代，再看模型计算，最后逐个理解并行机制。**

先区分两个名字：**Megatron-LM** 提供完整训练流程和示例；**Megatron Core** 是其中可以被其他训练框架复用的模型、并行和优化器组件，主要位于 `megatron/core/`。[官方说明](https://github.com/NVIDIA/Megatron-LM#about)

项目的主要结构如下，省略了辅助目录：

```text
Megatron-LM/
├── pretrain_gpt.py             # GPT 训练入口
├── model_provider.py          # 模型创建入口
├── gpt_builders.py             # GPT 组装逻辑
├── megatron/
│   ├── training/              # 训练流程
│   │   ├── training.py        # 初始化、训练循环、单步训练
│   │   ├── arguments.py       # 参数解析与校验
│   │   ├── initialize.py      # 运行环境初始化
│   │   └── checkpointing.py   # 训练状态保存与恢复
│   ├── core/                  # 可复用核心库
│   │   ├── models/            # GPT 等模型
│   │   ├── transformer/       # Transformer 层、Attention、MLP、MoE
│   │   ├── tensor_parallel/   # 张量并行
│   │   ├── pipeline_parallel/ # 流水线并行
│   │   ├── distributed/       # DDP、FSDP、梯度同步
│   │   ├── optimizer/         # 优化器及状态分片
│   │   ├── datasets/          # 数据集构建
│   │   ├── dist_checkpointing/# 分布式检查点
│   │   ├── fusions/           # 融合算子
│   │   ├── extensions/        # Transformer Engine 等后端适配
│   │   ├── inference/         # 推理
│   │   ├── parallel_state.py  # 并行通信组的创建与查询
│   │   └── process_groups_config.py # 通信组的组织与传递
│   └── legacy/                # 旧实现
├── examples/                  # 运行示例
├── tests/                     # 测试
└── tools/                     # 数据处理等工具
```

目录可以理解为：`training/` 管“训练如何推进”，`core/models/` 和 `core/transformer/` 管“模型如何计算”，其余核心模块解决“计算和状态如何分布到多张卡”。[项目结构](https://github.com/NVIDIA/Megatron-LM#project-structure)、[Core 目录](https://github.com/NVIDIA/Megatron-LM/tree/main/megatron/core)

建议按下面五步阅读。

1. **先读 `pretrain_gpt.py`，认清训练需要的几个接口。**

   重点找 `get_batch()`、`forward_step()`、`loss_func()` 和数据集 provider，以及文件底部对 `pretrain()` 的调用。回答三个问题：数据是什么格式？模型接收什么？损失如何计算？

   一个关键细节：`forward_step()` 返回的是**模型输出和损失计算函数**，反向传播由外层调度器安排。当前 GPT 构建逻辑还需要跟进 `model_provider.py` 和 `gpt_builders.py`。[训练入口](https://github.com/NVIDIA/Megatron-LM/blob/main/pretrain_gpt.py)、[GPT 构建代码](https://github.com/NVIDIA/Megatron-LM/blob/main/gpt_builders.py)

2. **进入 `training/training.py`，串起一次迭代。**

   先定位 `pretrain()`、`setup_model_and_optimizer()`、`train()`、`train_step()`。只追主路径，遇到日志、容错、性能统计等分支先标记下来。

   主要关系可以简化为：

   ```text
   pretrain_gpt.py
     └─ pretrain()
         ├─ 初始化运行环境
         ├─ 创建模型、优化器、学习率调度器
         ├─ 创建数据迭代器
         └─ train()
             └─ train_step()
                 ├─ 清理梯度
                 ├─ forward_backward_func(...)
                 │   ├─ 调用 forward_step() 完成前向
                 │   ├─ 计算 loss
                 │   └─ 安排反向传播与梯度同步
                 └─ optimizer.step()
   ```

   这里最值得理解的是：**训练循环、前后向调度、模型计算是三个不同层次。** 梯度通信还可能与反向计算重叠，不能只找一个孤立的同步调用。[训练主流程](https://github.com/NVIDIA/Megatron-LM/blob/main/megatron/training/training.py)

3. **沿 `GPTModel.forward()` 向下，看模型如何组成。**

   先建立这条计算路径：

   ```text
   GPTModel
     → Embedding
     → TransformerBlock
         → 多个 TransformerLayer
             → Attention + 残差
             → MLP / MoE + 残差
     → 输出投影
     → logits 或逐 token 的 loss
   ```

   优先读 `gpt_model.py`，再读 `transformer_block.py`、`transformer_layer.py`、`attention.py` 和 `mlp.py`。注意，`TransformerBlock` 在这里管理多层；开启流水线并行后，一个进程通常只持有其中一部分模型。[GPTModel](https://github.com/NVIDIA/Megatron-LM/blob/main/megatron/core/models/gpt/gpt_model.py)、[TransformerBlock](https://github.com/NVIDIA/Megatron-LM/blob/main/megatron/core/transformer/transformer_block.py)

   如果跳转定义时发现“实际执行的类”不直观，就查看 `gpt_layer_specs.py` 和 `spec_utils.py`。`ModuleSpec` 描述组件及其子模块，构建时才确定具体实现；所用配置可能选择本地实现，也可能选择 Transformer Engine 后端。[层配置](https://github.com/NVIDIA/Megatron-LM/blob/main/megatron/core/models/gpt/gpt_layer_specs.py)、[ModuleSpec](https://github.com/NVIDIA/Megatron-LM/blob/main/megatron/core/transformer/spec_utils.py)

4. **先理解 TP，再读 PP，随后读 DP 和优化器。**

   阅读并行代码时，始终问：**切了哪个维度？每张卡保存什么？何时需要通信？**

   | 机制 | 主要解决什么 | 源码阅读位置 |
   |---|---|---|
   | TP：张量并行 | 一层的矩阵计算分到多卡 | `tensor_parallel/` |
   | PP：流水线并行 | 不同层放在不同阶段 | `pipeline_parallel/` |
   | DP：数据并行 | 不同副本处理不同样本、同步梯度 | `distributed/` |
   | 分布式优化器 | 优化器状态等在 DP 组内分片 | `optimizer/` |
   | CP：上下文并行 | 长序列分到多卡计算 | 数据切分、Attention 后端及通信组 |
   | EP：专家并行 | MoE 专家分布在不同卡上 | `transformer/moe/` |

   SP（序列并行）也要留意：它通常配合 TP，把 LayerNorm、Dropout 等区域的激活沿序列维分片；CP 则涉及长序列 Attention 的分布式计算，两者需要分别理解。[并行机制说明](https://docs.nvidia.com/megatron-core/developer-guide/latest/user-guide/parallelism-guide.html)

   **TP 最适合从普通 MLP 入手。** 假设没有 SP，隐藏维度为 `H`，中间维度为 `4H`，TP 大小为 `p`：

   ```text
   输入                  [S, B, H]
   ColumnParallelLinear → 每卡 [S, B, 4H/p]
   激活函数              → 每卡独立计算
   RowParallelLinear    → 每卡产生部分结果，再求和
   输出                  [S, B, H]
   ```

   先读 `layers.py` 中的 `ColumnParallelLinear`、`RowParallelLinear`，再跟进通信和反向实现。前者按输出特征切分，后者按输入特征切分；“行/列”名称对应数学表达式中的矩阵，别直接套到 PyTorch 转置存储的权重维度上。[并行线性层](https://github.com/NVIDIA/Megatron-LM/blob/main/megatron/core/tensor_parallel/layers.py)、[MLP 实现](https://github.com/NVIDIA/Megatron-LM/blob/main/megatron/core/transformer/mlp.py)

   **PP 从 `schedules.py` 的三个调度函数读起：**

   ```text
   forward_backward_no_pipelining
     → forward_backward_pipelining_without_interleaving
     → forward_backward_pipelining_with_interleaving
   ```

   先理解单个 microbatch 的前向和反向，再看多个 microbatch 如何交错执行，最后跟进阶段之间的激活与梯度传输。[调度器源码](https://github.com/NVIDIA/Megatron-LM/blob/main/megatron/core/pipeline_parallel/schedules.py)

5. **用小配置验证理解，每次只增加一个变量。**

   建议从两层、短序列、普通 Dense GPT 开始，先设 `TP=PP=CP=1`，暂时关闭 MoE、重计算和通信重叠。随后依次尝试 `TP=2`、`PP=2`，再加入 DP。

   每到一个关键位置，记录四项：

   ```text
   当前 rank 和通信组
   输入、输出张量的 shape
   当前卡持有的参数切片
   forward / backward 中的通信操作
   ```

   配合 `tests/` 中对应模块的测试，观察构造参数和预期行为。你的第一阶段目标可以定得很具体：**能解释一个 microbatch 从 tokens 进入模型，到梯度产生、跨卡同步，再到参数更新的完整过程。** 这条路径读通后，再按需要深入 MoE、分布式检查点和性能优化。