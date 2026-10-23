# Action Tokenization & Latent Action Survey

A curated survey of action representations for robot learning, with emphasis on discrete action tokenizers, video-derived latent actions, and continuous-time trajectory representations. It focuses on how language, temporal structure, and executable robot control can be aligned.

> Coverage includes discrete action tokenization, video latent-action learning, hybrid discrete-token/continuous-action policies, cross-embodiment action spaces, and spline-based continuous policies.

## Contents

- [Full survey](trajectory_discretization_survey.md)
- [Discrete robot action tokenizers](trajectory_discretization_survey.md#一机器人动作序列的离散-tokenizer)
- [Discrete tokens with continuous action experts](trajectory_discretization_survey.md#二离散动作-token-与连续动作-expert-的混合接口)
- [Video latent actions](trajectory_discretization_survey.md#三视频-latent-action从无动作标注视频到可迁移控制)
- [Cross-embodiment action spaces](trajectory_discretization_survey.md#四跨-embodiment-的共享离散动作空间)
- [Continuous spline policies](trajectory_discretization_survey.md#五连续样条动作表示与离散-tokenizer-相邻的路线)
- [Comparison and research directions](trajectory_discretization_survey.md#综合比较与值得关注的方向)

## Scope

The survey prioritizes work that changes the action representation or its interface with policy learning and control. General trajectory-planning papers without a direct action-tokenization or policy connection are excluded from the main list.
