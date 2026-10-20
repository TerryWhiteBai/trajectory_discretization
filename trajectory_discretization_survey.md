# 轨迹离散化与控制优化相关论文调研

**检索范围：**以 2023–2026 年 arXiv 论文为主，重点关注 B-spline、傅里叶/DCT 等低维基函数或频域表示如何将连续轨迹/动作转成可优化、可控制或可预测的有限维变量。严格按今天（2026-10-17）往前三年计算，2023 年 10 月前的工作略早于窗口；仍保留少数 2023 年代表性工作作为邻近参考。下面按与你的“语言—时间—动作”离散表示研究的相关性分组。

## 一、与语言条件动作离散化最直接相关

### 1. FAST: Efficient Action Tokenization for Vision-Language-Action Models

**arXiv：** https://arxiv.org/abs/2501.09747

**简介核心工作：**提出 Frequency-space Action Sequence Tokenization（FAST），对机器人连续动作块做离散余弦变换（DCT），再对频域系数压缩/离散化，替代逐时间步、逐维度的直接分箱。方法利用动作序列的时间平滑性，减少高频动作建模所需 token 数，使自回归 VLA 能学习灵巧、高频控制；论文还发布在百万条真实动作轨迹上训练的 FAST+ 通用 tokenizer，并展示与 pi0 等策略结合后的扩展能力。对你的研究最直接的启发是：token 对应的是整段动作的频率结构，而不是孤立的时刻值；需关注截断系数、量化误差、反变换后的控制可执行性，以及语言监督如何进入 token 学习目标。

### 2. BEAST: Efficient Tokenization of B-Splines Encoded Action Sequences for Imitation Learning

**arXiv：** https://arxiv.org/abs/2506.06072

**简介核心工作：**提出 B-spline Encoded Action Sequence Tokenizer（BEAST），先用 B-spline 表示连续动作序列，再将样条参数编码为固定长度的离散或连续 token。与需要单独训练码本的 VQ/BPE tokenizer 不同，BEAST 利用样条系数直接产生统一长度的表示；解码后得到连续、平滑的控制信号。作者将表示接入 VAE、decoder-only Transformer 和 Florence-2 视觉语言模型，并在仿真与真实机器人任务中评估。它与“轨迹参数→token→连续动作控制”的研究问题高度重合，可重点比较其基函数系数是否保留动作语义、时间尺度和跨任务共享结构。

## 二、经典轨迹优化与控制中的 B-spline 参数化

### 3. An Optimization-Based Planner with B-spline Parameterized Continuous-Time Reference Signals

**arXiv：** https://arxiv.org/abs/2404.00133

**简介核心工作：**提出 BSPOP，以 B-spline 控制点参数化连续时间控制输入，针对规划器与低层控制器频率不匹配的问题生成任意采样频率下都可查询的参考信号。通过 B-spline 的凸包性质，可将连续控制输入约束在凸控制集合内；相较逐时刻离散优化，减少决策变量和不等式约束。仿真和实验结果显示，在较低计算开销下可达到高频基线相近的规划表现。这篇适合参考“参数数量—采样频率解耦—可行性约束”的方法设计。

### 4. PE-Planner: A Performance-Enhanced Quadrotor Motion Planner for Autonomous Flight in Complex and Dynamic Environments

**arXiv：** https://arxiv.org/abs/2403.12865

**简介核心工作：**面向复杂、动态和未知干扰环境中的高速四旋翼飞行，先在离散控制空间中做 kinodynamic 搜索，再用 B-spline 轨迹优化提升平滑度与障碍物间隙；局部规划器使用 model predictive contouring control（MPCC），联合考虑动力学、扰动、参考轨迹、控制/时间代价和安全约束生成实时控制。论文报告了仿真及真实飞行实验，包括复杂赛道上超过 6.8 m/s 的飞行。它展示了“低维样条轨迹优化 + 在线反馈控制”的组合范式。

### 5. Safe and Efficient Trajectory Optimization for Autonomous Vehicles using B-spline with Incremental Path Flattening

**arXiv：** https://arxiv.org/abs/2311.02957

**简介核心工作：**针对自动驾驶车辆中路径与速度耦合、车辆外形膨胀导致碰撞检测过保守的问题，提出盘形 swept-volume 估计、增量路径展平（IPF）和动力学可行性惩罚。优化变量由 B-spline 表示；通过碰撞点迭代推开轨迹，并逐步调整曲率代价，同时惩罚纵/横向速度和加速度以满足运动学、动力学限制。论文包含仿真和真实车辆实验。适合关注如何把几何轨迹基函数与路径—速度耦合、安全约束共同放进优化。

### 6. Spline-Based Minimum-Curvature Trajectory Optimization for Autonomous Racing

**arXiv：** https://arxiv.org/abs/2309.09186

**简介核心工作：**为自动赛车提出仅以 B-spline 控制点为优化变量的最小曲率轨迹优化方法，面向早期开发阶段车辆/赛道动力学数据有限的场景。相较离散轨迹点形式，样条表示提供连续性保证和局部可编辑性；作者报告在相近最优性下将决策变量维度降低约 90%，长赛道计算从秒级降至毫秒级。该工作发布于 2023 年 9 月，略早于严格三年窗口，但对“低维轨迹控制点作为动作表征”很有参考价值。

## 三、傅里叶表示：控制函数逼近与轨迹目标度量

### 7. Time Optimal Ergodic Search

**arXiv：** https://arxiv.org/abs/2305.11643

**简介核心工作：**研究机器人搜索中的时间最优遍历轨迹，将搜索覆盖质量写成基于余弦傅里叶谱系数的 ergodic metric，并把该指标的上界作为约束，以优化完成搜索所需时间。论文给出 Pontryagin 最优性条件和直接转录数值解，并在仿真及复杂环境中的无人机实验验证。注意：这里的傅里叶级数主要用于表示/度量空间覆盖分布，不是直接用傅里叶系数参数化整条控制轨迹；对“频谱特征如何进入控制目标”有启发，但与动作 tokenization 有区别。论文发表于 2023 年 5 月，略早于严格三年窗口。

### 8. A Multidimensional Fourier Approximation of Optimal Control Surfaces

**arXiv：** https://arxiv.org/abs/2306.16569

**简介核心工作：**用多维截断傅里叶级数逼近依赖初始条件与时间的最优控制函数和轨迹曲面，并结合改进的 augmented Lagrangian 将约束最优控制问题转为无约束优化。作者在牛顿力学的二次控制问题和复制子动力学博弈问题上验证方法，并讨论自动微分与傅里叶逼近误差界。它提供了“用少量频域系数表示一族控制解/轨迹曲面”的思路；与机器人单条轨迹控制不同，研究对象是控制/轨迹解的函数曲面。论文发表于 2023 年 6 月，略早于严格三年窗口。

## 归纳：对“语言—时间—动作”联合表示的可借鉴点

- **表示单元：**FAST 以频域系数压缩动作块；BEAST 以样条控制点表示连续动作；传统优化论文则直接优化样条控制点或谱特征。可以比较 token 是表示局部动作、整段运动形状，还是某个任务条件下的低维控制函数。
- **时间语义：**固定均匀采样的 DCT token 默认时间网格固定；B-spline 的 knot/control point 机制更容易表达非均匀时间分辨率和局部时间变化。若目标是共同对齐语言、时间和动作，可考虑把 knot/时长也作为显式变量或 token，而非只编码几何系数。
- **可执行性：**传统规划工作把动力学、障碍物、控制幅值/平滑性约束直接放入优化；动作 tokenizer 工作更强调重构与下游策略表现。一个潜在研究切口是联合训练语义对齐损失与动力学/可行性损失，并在解码轨迹上验证约束满足和闭环控制效果。
- **傅里叶与 B-spline 的差异：**傅里叶/DCT 对全局周期/频率结构和紧凑压缩有效；B-spline 具备局部支撑、控制点可解释性和连续性优势。对有接触、分阶段、局部突变或可编辑动作的任务，样条的局部性可能更合适；对高频但总体平滑的动作块，频域压缩可能更省 token。

> 注：上面只列出可在 arXiv 找到、且摘要/正文明确涉及样条、傅里叶/DCT 表示或由此驱动的轨迹/控制优化工作。标题和简介以论文页面摘要及正文为依据；“严格三年窗口外”的 2023 年 5–9 月论文已在条目中注明，引用时请按你的论文时间范围决定是否保留。

## 补充检索（2024–2026）

以下是第二轮检索新增的论文，按其研究对象拆分到“连续样条策略”和“相关基函数/任务驱动规划”两类。

## 四、连续 B-spline 动作策略与自适应时间表示

### 9. B-spline Policy: Accelerating Manipulation Policies via B-spline Action Representations

**arXiv：** https://arxiv.org/abs/2607.09648

**简介核心工作：**提出 B-spline Policy（BSP），策略直接预测样条的 knot 与控制点，以连续 B-spline 曲线取代固定长度、固定离散时间间隔的动作块。解码动作可按不同频率查询，并通过时间缩放支持高速执行；作者将表示接入现有策略学习管线，在仿真和真实操作任务中考察成功率与任务完成时间。与 BEAST 的离散 tokenizer 路线相比，BSP 更强调连续参数预测、时间缩放和策略执行效率，适合研究动作 token 是否需要显式携带时长与 knot 位置。

### 10. Spline Policy: A Structured Representation for Robot Policies

**arXiv：** https://arxiv.org/abs/2606.07386

**简介核心工作：**提出 Spline Policy（SP），将常见固定分辨率动作块替换成样条参数，同时保持策略骨干不变。样条可连续解码、任意时间分辨率查询、在参数空间编辑或约束，并传递给下游控制器；作者还针对二次样条构造状态相关向量场，用于围绕生成运动的局部纠正，并分析从观测不确定性传播到样条和轨迹的不确定性。方法覆盖 diffusion、flow-matching、Transformer 与 VLA 骨干，并在仿真和真实机器人场景验证。这篇将表示层与经典控制机制连接起来，尤其值得参考其参数空间编辑和执行时纠偏设计。

### 11. SplineWAM: Adaptive Action Horizons for World Action Models via B-Spline Representations

**arXiv：** https://arxiv.org/abs/2609.39873

**简介核心工作：**针对 World Action Model 每次都输出固定长度动作块的问题，用固定参数预算的三次 B-spline 表示动作，并拟合非均匀 knot 时间，使不同动作复杂度获得不同时间分辨率和执行时长。视频监督也对齐到拟合后的 knot 时间；异步执行时用 Jacobian-Pullback Real-Time Chunking 校正样条参数，使已提交的动作前缀在解码后保持连续。作者在 LIBERO-Plus、RoboCasa 及双臂实机任务上评估。它直接把动作复杂度、时间分辨率和模型调用节奏联系起来，与你关注的时间信息对齐尤其相关。

## 五、相邻的基函数规划与任务驱动轨迹优化

### 12. Smooth trajectory generation and hybrid B-splines-Quaternions based tool path interpolation for a 3T1R parallel kinematic milling robot

**arXiv：** https://arxiv.org/abs/2602.16758

**简介核心工作：**面向并联加工机器人，将 B-spline 用于位置路径、四元数用于姿态插值，并以 Bezier 曲线拟合路径弧长与工具姿态之间的非线性关系，再用序列二次规划求解。时间参数化分两阶段优化最小 jerk 与时间最优分段曲线，并在任务空间、关节空间依次实施。工作重点是加工轨迹规划和插补，不是学习式动作 tokenization；可作为位置—姿态—时间分解及异构轨迹分量联合参数化的参考。

### 13. Observability-Aware Active Calibration of Multi-Sensor Extrinsics for Ground Robots via Online Trajectory Optimization

**arXiv：** https://arxiv.org/abs/2506.13420

**简介核心工作：**将 B-spline 控制点用于地面机器人在线轨迹优化，以 Fisher 信息矩阵最小特征值衡量多传感器外参的可观测性，并主动规划/重规划轨迹来改善标定数据质量。传感器包括麦克风阵列、LiDAR 和轮速编码器；论文提供仿真与实机验证。此处轨迹参数化服务于“信息采集/可观测性”目标，而非最短路或动作重构，适合参考任务语义如何转化为轨迹优化代价。

### 14. Efficient and Real-Time Motion Planning for Robotics Using Projection-Based Optimization

**arXiv：** https://arxiv.org/abs/2506.14865

**简介核心工作：**提出增广拉格朗日谱投影梯度方法（ALSPG），通过欧氏投影、Minkowski 和及基函数表达几何约束，求解机器人运动规划问题。作者强调使用几何投影与基函数提升实时求解效率，并在 Franka、P-Rob 机械臂和小车上展示实验。论文摘要将 basis functions 纳入规划优化，但并非专门针对 B-spline 或傅里叶动作 token；可作为“低维基函数 + 几何可行域投影”优化范式的相关工作。

## 跨类别比较与研究启发

- **与你的表示学习问题最贴近：**FAST（DCT 系数离散 token）、BEAST（样条编码动作 token）、BSP/SP（直接预测连续样条参数）、SplineWAM（非均匀 knot 与自适应动作时域）。这几类工作可按“频域离散 token / 样条离散 token / 连续样条参数”建立清晰对照。
- **时间信息显式化程度：**FAST 通常作用于固定采样动作块；BEAST 用固定长度表示；BSP/SP 可重采样与时间缩放；SplineWAM 进一步预测非均匀 knot、可变执行时长和策略下一次调用时刻。若论文主张对齐时间，后两类尤其值得精读。
- **优化控制侧补充：**BSPOP、PE-Planner、自动驾驶 B-spline、主动标定轨迹优化等把控制/安全/信息目标直接作用到有限维基函数参数上；它们能补足 VLA tokenizer 文献通常没有严格动力学可行性分析的问题。
- **检索边界：**新增条目仍以 arXiv 论文为主。标题含“basis”但未明确指 B-spline/Fourier 的论文已标注为广义相关，不应与直接采用这些基函数的工作等同引用。

## 全部论文的分类总览

为便于写 related work，可以将上面 14 篇按研究问题归并如下（编号对应前文条目）：

| 类别 | 论文编号 | 表示对象与主要目标 |
| --- | --- | --- |
| 离散动作序列 tokenization | 1 FAST；2 BEAST | 将连续动作块变成频域或样条参数 token，服务于 VLA/序列模型训练与动作压缩。 |
| 连续样条动作策略 | 9 BSP；10 Spline Policy；11 SplineWAM | 策略预测样条参数；重点是连续解码、可变采样率/时域、时间缩放或异步动作块执行。 |
| B-spline 轨迹优化与控制 | 3 BSPOP；4 PE-Planner；5 自动驾驶 B-spline；6 赛车最小曲率 | 以控制点作为低维优化变量，将平滑、动力学、安全、曲率或实时控制约束作用于轨迹。 |
| 傅里叶/谱域控制与轨迹目标 | 7 Time Optimal Ergodic Search；8 Multidimensional Fourier Approximation | 7 以傅里叶谱度量遍历覆盖质量并优化搜索时间；8 用傅里叶基逼近最优控制/轨迹曲面。二者都不是离散动作 tokenizer。 |
| 相邻的任务专用基函数规划 | 12 加工机器人 B-spline/四元数插补；13 主动标定轨迹优化；14 投影式实时运动规划 | 基函数服务于姿态/时间插补、信息可观测性或几何可行域投影，作为控制优化背景文献更合适。 |

### 综述叙事建议

可以先介绍经典最优控制如何用低维基函数替代逐时刻决策变量（类别 3），再转向学习式动作表示：频域/样条离散 token（类别 1）和连续样条策略（类别 2）。随后讨论时间变量的显式化程度，从固定时间网格到时间缩放、非均匀 knot 和自适应动作时域；最后指出语义对齐和动力学可行性仍常由不同模块处理，构成联合学习语言、时间结构与可执行动作表示的研究空间。傅里叶控制曲面与遍历度量（类别 4）可作为频域表示脉络，但需明确其优化目标和表示对象与 VLA tokenization 不同。
