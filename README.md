# Action Tokenization & Latent Action Survey

A curated survey of action representations for robot learning, focusing on discrete action tokenizers, video-derived latent actions, hybrid discrete-token/continuous-action policies, cross-embodiment representations, and spline-based continuous policies.

> The goal is to understand how language, temporal structure, and executable robot control can be aligned through action representations.

---

## Contents

- [Discrete Robot Action Tokenizers](#discrete-action-tokenizers)
- [Discrete Tokens with Continuous Action Experts](#hybrid-discrete-continuous)
- [Video Latent Actions](#video-latent-actions)
- [Cross-Embodiment Action Spaces](#cross-embodiment)
- [Continuous Spline Policies](#continuous-spline-policies)
- [Related Survey](#related-survey)
- [Comparison and Research Directions](#comparison-and-directions)

---

**范围与筛选：**聚焦近年机器人/VLA 中的离散动作 tokenizer、视频 latent action、跨 embodiment 动作码，以及直接参数化连续动作的样条表示。移出与动作表示关系较弱的通用轨迹优化、赛车路径规划、遍历搜索和加工轨迹等背景工作。以下论文主要为 2023–2026 年 arXiv 预印本；论文简介概括其表示设计与主要目标。

<a id="discrete-action-tokenizers"></a>

## 一、机器人动作序列的离散 tokenizer

这一类以已有机器人动作序列为输入，学习离散 code/token，再由策略预测 token 并解码为连续控制。主要研究问题从压缩和重构，逐渐扩展到语义对齐、策略可预测性、物理关系保持和状态条件解码。


### FAST: Efficient Action Tokenization for Vision-Language-Action Models

**arXiv：** https://arxiv.org/abs/2501.09747

**简介核心工作：**提出 Frequency-space Action Sequence Tokenization（FAST），对机器人连续动作块做离散余弦变换（DCT），再对频域系数压缩/离散化，替代逐时间步、逐维度的直接分箱。方法利用动作序列的时间平滑性，减少高频动作建模所需 token 数，使自回归 VLA 能学习灵巧、高频控制；论文还发布在百万条真实动作轨迹上训练的 FAST+ 通用 tokenizer，并展示与 pi0 等策略结合后的扩展能力。对你的研究最直接的启发是：token 对应的是整段动作的频率结构，而不是孤立的时刻值；需关注截断系数、量化误差、反变换后的控制可执行性，以及语言监督如何进入 token 学习目标。

### BEAST: Efficient Tokenization of B-Splines Encoded Action Sequences for Imitation Learning

**arXiv：** https://arxiv.org/abs/2506.06072

**简介核心工作：**提出 B-spline Encoded Action Sequence Tokenizer（BEAST），先用 B-spline 表示连续动作序列，再将样条参数编码为固定长度的离散或连续 token。与需要单独训练码本的 VQ/BPE tokenizer 不同，BEAST 利用样条系数直接产生统一长度的表示；解码后得到连续、平滑的控制信号。作者将表示接入 VAE、decoder-only Transformer 和 Florence-2 视觉语言模型，并在仿真与真实机器人任务中评估。它与“轨迹参数→token→连续动作控制”的研究问题高度重合，可重点比较其基函数系数是否保留动作语义、时间尺度和跨任务共享结构。

### VQ-VLA: Improving Vision-Language-Action Models via Scaling Vector-Quantized Action Tokenizers

**arXiv：** https://arxiv.org/abs/2507.01016

**简介核心工作：**提出基于卷积 residual VQ-VAE 的机器人动作 tokenizer，并研究扩大 tokenizer 训练数据规模的效果。工作利用大规模真实与合成轨迹数据学习时空动态 codebook，报告更平滑连贯的动作输出、较快推理，以及在短时反应和长时规划任务上的迁移表现。作者指出合成轨迹与真实轨迹的差距相对有限，可用于扩展 tokenizer 数据。它与 FAST/BEAST 同属有动作标签序列的离散化路线，主要差别是学习式向量量化和数据规模扩展。

### Action Tokenizer Matters in In-Context Imitation Learning

**arXiv：** https://arxiv.org/abs/2503.01206

**简介核心工作：**系统比较多种动作 tokenizer 在 in-context imitation learning（ICIL）中的效果，发现重构准确并不保证解码动作足够平滑。作者提出 LipVQ-VAE，通过权重归一化在 latent action 空间施加 Lipschitz 约束，使相近输入对应更平滑的潜变量/解码轨迹，并报告模拟与真实机器人实验中的稳定性收益。启示是把时间平滑、局部连续性和闭环可执行性纳入 tokenizer 质量指标，而不只报告离线重构误差。

### ActionPiece: Rethinking Action Tokenization for Autoregressive Vision-Language-Action Models

**arXiv：** https://arxiv.org/abs/2609.18487

**简介核心工作：**指出逐点重构误差未必能衡量 tokenizer 是否保留不同示范动作之间的物理关系，提出 physical rank consistency（PRC）评价解码动作对局部动作距离排序的保持程度，并以此联合监督表示学习与量化，形成离散 action tokens。工作强调 tokenization 不仅要还原单条轨迹，也应保留动作调整的相对关系，以支持策略在不同情境中选择合适动作。

### ActionCodec: What Makes for Good Action Tokenizers

**arXiv：** https://arxiv.org/abs/2602.15397

**简介核心工作：**从 VLA 优化而非单纯重构角度提出 tokenizer 设计原则，包括提高时间 token overlap、减少词表冗余、增强多模态互信息和 token 独立性，并据此构造 ActionCodec。论文报告在多个仿真和真实任务上的训练效率及策略表现提升，强调 tokenizer 应为下游策略提供易学习的序列目标。

### Beyond Reconstruction: What Matters for Action Tokenization for Robot Policies?

**arXiv：** https://arxiv.org/abs/2610.09170

**简介核心工作：**提出 Predictable and Robust Action Tokenization（ProAct），在动作重构目标之外加入面向下游预测性和鲁棒性的训练设计。论文指出，tokenizer 不仅要重构训练轨迹，还要让策略容易预测正确 token，并确保未见 token 序列解码后仍是合理动作；方法仅用动作数据训练，可适配多种 tokenizer 架构，并在模拟、VLA 和实机评估中报告收益。

### SA-VLA: State-aware tokenizer for improving Vision-Language-Action Models' performance

**arXiv：** https://arxiv.org/abs/2606.30113

**简介核心工作：**提出 state-aware action tokenizer，在离散 token 解码时显式条件化机器人本体状态。作者比较 state-action cross-attention 和轻量 state adapter；后者通过调制因子使一个离散 code 对应一族随状态变化的连续动作，从而缓解固定 code prototype 的压缩误差，并兼容自回归或并行 token 解码。

### Behavior-Aligned Action Tokenization for Robot Policy Learning

**arXiv：** https://arxiv.org/abs/2609.27513

**简介核心工作：**提出 BAAT，用 Soft-DTW 在不同示范中寻找对应动作片段，并联合重构与量化坐标对齐，使执行方式相似但时间不同的片段获得相近 token 表示。下游自回归策略预测离散 token，再由 history-conditioned diffusion decoder 还原连续动作；工作强调用行为对应关系监督离散码本，而不只靠逐点重构。

### X-Tokenizer: A Multimodal Action Tokenizer for Vision-Language-Action Pretraining

**arXiv：** https://arxiv.org/abs/2606.14752

**简介核心工作：**提出面向 VLA 预训练的多模态动作 tokenizer，将 tokenization 视为语义推理与可执行控制之间的接口。Semantic Residual Quantization 的首层通过 masked action modeling 编码粗粒度动作意图，后续残差层保留精细运动信息；再用与预训练基础模型的对比对齐和下一帧视觉语言特征预测增强语义 grounding。作者报告在 2.4M 条轨迹（约 2B action frames）上预训练，并在多模态 grounding 和长时程任务上优于 FAST。

<a id="hybrid-discrete-continuous"></a>

## 二、离散动作 token 与连续动作 expert 的混合接口

近期 VLA 开始把“离散 token”与“连续动作生成”拆成互补角色：token 可以为 VLM 提供动作结构化监督或压缩后的计划表示，而 flow/diffusion expert 仍在连续空间生成可执行轨迹。要区分两种设计：有的工作在推理时真正由连续 expert 出动作；有的仍由自回归 expert 生成离散 token，再经 tokenizer 解码。两者都在探索 token 化与连续控制的协同，但不是同一种融合。

### Ordered Action Tokens for Visuomotor Policy Learning (OAT)

**arXiv：** https://arxiv.org/abs/2607.21670

**简介核心工作：**提出 Ordered Action Tokenization（OAT），用 register transformer、Finite Scalar Quantization 和 prefix 训练，将动作块编码为有序 token：早期 token 提供粗粒度且可解码的整段动作，后续 token 逐步细化残差。论文验证两种使用方式：自回归策略直接生成 token 并解码；以及 token co-training——离散动作 token loss 训练 VLM，VLM 的视觉/语言/状态上下文再条件化 flow-matching action expert，由 expert 在推理时生成连续动作。后者是本组论文中最直接的“离散监督 + 连续动作专家”融合方案，也让 token 负责可预测的动作计划表征、连续 expert 负责高保真控制。

### FASTer: Toward Powerful and Efficient Autoregressive Vision–Language–Action Models with Learnable Action Tokenizer and Block-wise Decoding

**OpenReview：** https://openreview.net/forum?id=k6nTUFoqeT  ·  **arXiv：** https://arxiv.org/abs/2512.04952

**简介核心工作：**提出 FASTerVQ，以动作 patchification、Transformer/RVQ 及频域与时域重建损失压缩连续动作块；FASTerVLA 再以轻量 action expert 和 block-wise autoregressive 解码生成离散 action tokens，最后由 VQ decoder 还原连续轨迹。它与 OAT 的 token co-training 不同：FASTer 的 expert 仍输出离散码，而不是 flow/diffusion 连续动作；其创新点是让可学习 tokenizer、动作专用 expert 和分块并行解码协同，改善压缩、推理延迟及跨 embodiment 泛化。

### ReAT: Residual Action Tokenization for Autoregressive Robot Policies

**arXiv：** https://arxiv.org/abs/2610.13093

**简介核心工作：**提出 Residual Action Tokenization（ReAT），每个新 token 根据当前已解码动作与目标动作之间的残差构造，并通过 reconstruction-conditioned decoder 显式修正前缀重建。任意非空 token 前缀都能解码为完整动作块，因此可按 token budget 渐进式提高动作精度。ReAT 是离散 token 与连续重建之间更紧密的闭环协同，但其策略仍预测离散 token、解码器还原动作；不是独立的连续 flow/diffusion expert。它与 OAT 的 ordered prefix 互补：OAT强调前缀次序及粗到细信息分配，ReAT明确让后续码对当前重建误差负责。

**融合脉络：**FASTer 优化“离散 action expert → tokenizer 解码”的效率；ReAT 让每个离散码对应可解释的连续轨迹修正；OAT 则进一步验证“离散 token 监督 VLM + 连续 flow expert 执行”的异构双头路线。研究语言—时间—动作对齐时，可把 token 用作语言可预测、具时间顺序的高层动作计划，同时让连续 expert 负责精细、平滑且满足控制约束的轨迹；核心问题包括 token loss 与 flow loss 的梯度/语义对齐、专家上下文是否包含离散动作前缀、以及两路输出在闭环执行中的互补性。

<a id="video-latent-actions"></a>

## 三、视频 latent action：从无动作标注视频到可迁移控制

这条路线从视频帧间变化中推断动作变量。关键进展不是单纯压缩视频，而是让 latent action 能解释状态转移，并进一步被策略预测、映射为机器人控制或用于规划。需注意：视频 latent 往往先编码“画面如何变化”，并不天然等价于特定机器人的可执行动作。

### 概念奠基：从无标签视频学习可控的潜在动作

#### Genie: Generative Interactive Environments

**arXiv：** https://arxiv.org/abs/2402.15391

**简介核心工作：**提出可交互生成环境 Genie，从无标注互联网视频学习可控的环境模型。系统由视频 tokenizer、自回归 dynamics model 和 latent action model 构成；模型从相邻帧变化中发现潜在动作，并允许用户用潜在动作控制生成环境。其突破在于证明无需动作标注也能从视频中学出有用的交互变量，是视频 latent action / generative world model 的代表性起点；但它主要在 2D 游戏环境验证，不等于直接学得机器人控制器。

#### Learning to Act without Actions

**arXiv：** https://arxiv.org/abs/2312.10812

**简介核心工作：**提出 Latent Action Policies（LAPO），从视频观测中恢复潜在动作信息，并据此训练 latent-action policy、world model 和 inverse dynamics model，不依赖真实动作标签。论文展示 latent policy 可通过少量动作标注离线微调，或通过奖励在线适配。它把无动作标签视频学习从生成式交互环境推进到策略学习，是后续机器人视频 latent-action 工作的重要方法基础。

### 视频预训练迁移到机器人：从 token 预测到物理 grounding

#### Latent Action Pretraining from Videos (LAPA)

**arXiv：** https://arxiv.org/abs/2410.11758

**简介核心工作：**提出 LAPA，用 VQ-VAE 式量化器从相邻视频帧学习离散 latent actions，再预训练 VLA 根据当前观察和任务描述预测这些 tokens，最后用少量机器人动作数据将 latent action 映射到真实控制。重点是利用无动作标签的人类/网络视频扩大 VLA 预训练数据，建立“视频变化 token → 机器人动作”的迁移链路。

#### Moto: Latent Motion Token as the Bridging Language for Learning Robot Manipulation from Videos

**arXiv：** https://arxiv.org/abs/2412.04445

**简介核心工作：**通过 VQ-VAE 式 Latent Motion Tokenizer 将相邻视频帧间运动编码成离散 motion tokens，再用 Moto-GPT 自回归预测 token 序列，从视频中学习运动先验。随后以 co-fine-tuning 将 latent motion token 预测与低层机器人动作预测连接起来，把 motion token 作为视频运动和机器人执行之间的桥接表示。

#### villa-X: Enhancing Latent Action Modeling in Vision-Language-Action Models

**arXiv：** https://arxiv.org/abs/2507.23682

**简介核心工作：**提出 Visual-Language-Latent-Action（ViLLA）框架。latent action model 联合 inverse dynamics、视觉 forward dynamics 和 proprioceptive forward dynamics：从前后帧推断 latent action，并预测未来视觉变化与机器人状态/动作；VLA actor 再联合预测 latent actions 和低层机器人动作。proprioceptive dynamics 将视频 latent 与机器人状态及物理动作联系起来，强化从人类视频到机器人控制的 grounding。

### 扩展方向：latent-action world model、适应性与系统评估

#### AdaWorld: Learning Adaptable World Models with Latent Actions

**arXiv：** https://arxiv.org/abs/2503.18938

**简介核心工作：**从视频自监督抽取 latent actions，并训练以 latent action 为条件的自回归 world model，以减少对大规模显式动作标注的依赖。论文重点展示该表示如何帮助模型在新环境中通过少量交互适配新动作，并用于视觉规划；它把 latent action 从表征/预训练目标推进到 world model 的控制接口与适应机制。

#### Latent Action Pretraining Through World Modeling (LAWM)

**arXiv：** https://arxiv.org/abs/2509.18428

**简介核心工作：**提出 LAWM，通过未来状态预测的 world-model 目标从无标签机器人或人类视频中学习 latent action，再迁移到 imitation learning。它强调让 latent action 对状态转移具有解释力，并检验跨任务、环境和 embodiment 的迁移。与 LAPA 的 token 预测式 VLA 预训练相比，LAWM 更突出动力学建模信号。

#### Learning Latent Action World Models In The Wild

**arXiv：** https://arxiv.org/abs/2601.05230

**简介核心工作：**将 latent-action world model 扩展到多样化的 in-the-wild 视频，系统处理噪声、视角差异和缺少共同 embodiment 等问题。作者发现受约束的连续 latent action 比常见向量量化更能表达复杂视频动作；在缺少统一 embodiment 时，latent 主要描述相机坐标下的局部运动，但仍可学习控制到 latent 的映射并支持规划。这是离散 codebook 主线的重要连续表示对照。

#### What Matters for Latent Actions in Robot Learning

**arXiv：** https://arxiv.org/abs/2608.19613

**简介核心工作：**在统一 autoencoding 框架下，对机器人操作 latent action model 的建模范式、学习目标、正则化和集成策略进行系统实证比较，并研究 latent 质量代理指标能否预测下游任务表现。它不是新 tokenizer，而是帮助理解不同 latent-action 设计何时有效的比较基准，适合作为实验设计与评价依据。

**主线梳理：**Genie 展示从无标签视频发现可控 latent 的可能性；LAPO 把 latent action 接入策略和逆动力学学习；LAPA、Moto、villa-X 则分别推进 VLA 预训练、运动 token 桥接及机器人状态 grounding；AdaWorld、LAWM 和 in-the-wild work 把研究扩展到 action-conditioned world model、快速适应与更复杂视频分布。

<a id="cross-embodiment"></a>

## 四、跨 embodiment 的共享离散动作空间

该路线学习通用动作词表，并将 token 解码到不同机器人的控制接口，重点处理动作空间异构。


### Universal Actions for Enhanced Embodied Foundation Models

**arXiv：** https://arxiv.org/abs/2501.10105

**简介核心工作：**提出 UniAct 的 universal action space，以离散向量量化 code 表示不同机器人共享的原子行为，并通过 embodiment-specific 解码将通用动作还原为各自控制接口。目标是缓解跨机器人数据中关节、末端执行器及控制频率不一致导致的动作空间异构，支持跨 embodiment 训练和迁移。它学习跨机器人示范的共享离散动作词表，而非从视频无监督发现动作。

<a id="continuous-spline-policies"></a>

## 五、连续样条动作表示：与离散 tokenizer 相邻的路线

这些工作直接预测连续样条参数，而非离散词表；保留在调研中作为结构化轨迹表示的对照。它们尤其关注连续时间查询、时间缩放和可变动作时域。


### B-spline Policy: Accelerating Manipulation Policies via B-spline Action Representations

**arXiv：** https://arxiv.org/abs/2607.09648

**简介核心工作：**提出 B-spline Policy（BSP），策略直接预测样条的 knot 与控制点，以连续 B-spline 曲线取代固定长度、固定离散时间间隔的动作块。解码动作可按不同频率查询，并通过时间缩放支持高速执行；作者将表示接入现有策略学习管线，在仿真和真实操作任务中考察成功率与任务完成时间。与 BEAST 的离散 tokenizer 路线相比，BSP 更强调连续参数预测、时间缩放和策略执行效率，适合研究动作 token 是否需要显式携带时长与 knot 位置。

### Spline Policy: A Structured Representation for Robot Policies

**arXiv：** https://arxiv.org/abs/2606.07386

**简介核心工作：**提出 Spline Policy（SP），将常见固定分辨率动作块替换成样条参数，同时保持策略骨干不变。样条可连续解码、任意时间分辨率查询、在参数空间编辑或约束，并传递给下游控制器；作者还针对二次样条构造状态相关向量场，用于围绕生成运动的局部纠正，并分析从观测不确定性传播到样条和轨迹的不确定性。方法覆盖 diffusion、flow-matching、Transformer 与 VLA 骨干，并在仿真和真实机器人场景验证。这篇将表示层与经典控制机制连接起来，尤其值得参考其参数空间编辑和执行时纠偏设计。

### SplineWAM: Adaptive Action Horizons for World Action Models via B-Spline Representations

**arXiv：** https://arxiv.org/abs/2609.39873

**简介核心工作：**针对 World Action Model 每次都输出固定长度动作块的问题，用固定参数预算的三次 B-spline 表示动作，并拟合非均匀 knot 时间，使不同动作复杂度获得不同时间分辨率和执行时长。视频监督也对齐到拟合后的 knot 时间；异步执行时用 Jacobian-Pullback Real-Time Chunking 校正样条参数，使已提交的动作前缀在解码后保持连续。作者在 LIBERO-Plus、RoboCasa 及双臂实机任务上评估。它直接把动作复杂度、时间分辨率和模型调用节奏联系起来，与你关注的时间信息对齐尤其相关。

<a id="related-survey"></a>

## 六、综述与分类框架


### A Survey on Vision-Language-Action Models: An Action Tokenization Perspective

**arXiv：** https://arxiv.org/abs/2507.01925

**简介核心工作：**从 action tokenization 角度综述 VLA，将动作 token 分为语言描述、代码、affordance、轨迹、目标状态、latent representation、原始动作和 reasoning 等类别，并讨论不同 token 类型的优势、数据需求与 grounding 路径。它不是新 tokenizer 方法，但适合定位轨迹 token 在整体 action-token 谱系中的位置。

<a id="comparison-and-directions"></a>

## 综合比较与值得关注的方向

| 路线 | 代表工作 | 表示特点 / 主要突破 |
| --- | --- | --- |
| 频域/结构化压缩 | FAST、BEAST | 用 DCT 或样条参数代替逐时刻分箱，压缩动作序列并保留平滑结构。 |
| 大规模 VQ 动作词表 | VQ-VLA | 通过扩展真实与合成轨迹数据提升 codebook 覆盖和跨任务复用。 |
| 离散 token + 连续 expert | OAT；相邻路线：FASTer、ReAT | OAT 用离散 token loss 训练 VLM、flow expert 生成连续动作；FASTer 和 ReAT 仍由 token expert 生成离散码并解码，但将 tokenizer、action expert 与连续重建更紧密协同。 |
| 面向策略训练的 tokenizer | ActionCodec、ProAct | 直接考虑 token 序列是否容易被策略学习、预测，及未见 token 的解码鲁棒性。 |
| 物理/行为结构保持 | LipVQ-VAE、ActionPiece、BAAT | 分别强调时间平滑、动作关系排序、跨示范行为对应。 |
| 状态/语义条件化解码 | SA-VLA、X-Tokenizer | 前者让离散码按机器人状态适配连续控制；后者将粗动作意图与残差细节分层并对齐视觉语言语义。 |
| 视频 latent action | Genie、LAPO、LAPA、Moto、villa-X、AdaWorld、LAWM | 从视频发现可控 latent，并逐步连接策略学习、VLA 预训练、机器人状态 grounding 与 world-model 规划。 |
| 通用跨 embodiment 动作 | UniAct | 学习可通过 embodiment-specific decoder 落地的共享原子动作 code。 |
| 连续样条策略 | B-spline Policy、Spline Policy、SplineWAM | 不使用离散词表，显式保留连续时间、knot 和动作时域结构。 |

### 与“语言—时间—动作”对齐的研究启发

- **从压缩接口转向语义接口：**X-Tokenizer 将动作意图与细节残差分层，并对齐预训练视觉语言特征；OAT 展示离散 token 还可作为 VLM 的动作监督，交由连续 flow expert 执行。可进一步研究 token 是否对应可命名的动作阶段或语言片段。
- **把策略可学习性纳入 tokenizer 目标：**ActionCodec 与 ProAct 的重点是 tokenizer 产出的序列是否利于策略优化、预测，以及对分布外 token 是否稳健，不只看重构误差。
- **显式建模时间变化：**BAAT 对齐跨演示中时间不同但行为对应的片段；样条策略和 SplineWAM 则显式表示 knot 或可变执行时域。可将动作边界、时长和非均匀时间采样作为 token 的一部分。
- **区分视觉 latent 与控制 latent：**Genie/LAPA/Moto 等从视频学习的 latent 可能只描述图像变化；状态条件解码（SA-VLA）或本体动力学监督（villa-X）提供了建立控制语义的重要路径。
- **可执行性评价：**建议同时报告 token 率、重构误差、语义/行为对应、时间平滑、分布外解码有效性、闭环成功率和跨 embodiment 迁移。

> 筛选说明：本版移除了 B-spline 赛车/自动驾驶/无人机规划、傅里叶遍历搜索及加工路径等通用轨迹优化论文；它们可另列为控制背景文献，但与 action tokenizer 主线关联较弱。
