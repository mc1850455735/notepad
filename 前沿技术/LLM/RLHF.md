# 1. 大模型里应用强化学习时，状态，动作空间，动作都是什么



# 2. RLHF 和 SFT 有什么区别？为什么只做 SFT 不够？

## (1) RLHF 和 SFT 的区别

**训练目标不同**
- SFT：让模型学会像标注答案一样输出，目标是拟合人工示范分布；
- RLHF：让模型学会输出更符合人类偏好的答案，目标是在 RM 提供的奖励信号下，优化模型输出，使其更符合人类偏好。

**训练信号不同**
- SFT：使用的是标准监督信号，即给定 prompt 和 reference answer，最小化 next-token prediction loss，依赖示范答案；
- RLHF：使用的是偏好信号，即人类比较两个或多个回答哪个更好，再由 RM 给出奖励，PPO 等算法据此优化策略，依赖奖励信号。

**优化方式不同**
- SFT：本质是监督学习，直接拟合人工示范数据；
- RLHF：本质是强化学习，在 RM 提供的奖励下优化生成策略，通常还会加 KL 约束，限制策略漂移、保持训练稳定性并降低 reward hacking 风险。

**对回答的评价方式不同**
- SFT：假设训练集中给出的示范就是目标输出；
- RLHF：不要求只有唯一标准答案，而是通过偏好比较学习哪个更好。

## (2) 为什么只做 SFT 不够

**示范数据只能使模型学会基础回复规律，但是很难做到答得好**
通过 SFT 的训练，能教会模型基本的任务跟随和示范模仿能力，但是很多现实任务没有唯一标准答案，但是其质量存在高低。同时，人类还会在意其他因素，如答案的帮助性、真实性、安全性、简洁性、礼貌性等。这些因素很难靠单独的 reference answer 表达。

**SFT 容易产生平均化回答**
SFT 本质上是使模型拟合示范分布，模型在开放式任务中输出往往倾向于更保守、模板化或泛化过强的倾向，未必总能贴合具体用户偏好。通过 RLHF，可以进一步把模型往更有帮助、更自然、更符合人类主观评价的方向进行训练。

**很多对齐目标不是 token-level supervision 能直接表达的**
SFT 通过最小化 next-token prediction loss 的方式进行模型训练，而在对齐阶段，很多目标无法在 token-level 体现，如：是否啰嗦、是否语气得体、是否兼顾帮助性与安全性等。这些通常更适合通过整体回答级别的偏好/奖励来建模，而不是逐 token 地监督。

**SFT 对分布外场景和长尾偏好适应有限**
SFT 主要学习 “训练集中别人怎么写”，对训练分布内的常见模式学习效果较好。但当用户问题更开放、更复杂、更主观时，仅靠模仿示范往往不足；对于一些长尾偏好或难以通过标准答案枚举的行为要求，RLHF 可以进一步通过偏好信号修正模型行为。

# 3. RLHF训练时，Reward Model和LLM是同时训练还是先后训练，instruct GPT论文里是如何训练RM的

## (1) RM 和 LLM 的训练顺序

InstructGPT 里，RM 和 LLM 是分阶段先后训练的。通常先对预训练 LLM 做 SFT，再训练 RM；最后固定 RM，用 RM 和 PPO 继续优化 LLM。

其经典流程分为三步：
1. 监督微调 ( SFT )：先用人工编写的高质量示范数据，对预训练 LLM 做监督微调，得到一个 SFT policy；
2. 训练 Reward Model (RM)：固定一批模型生成结果，让标注员对多个回答进行偏好排序，基于这些偏好数据训练 RM。RM 在训练完成后，通常作为固定模型用于 PPO，不再进行更新；
3. 用 RM 对 LLM 做强化学习：以 SFT 模型为初始化策略，用 RM 作为奖励函数，通过 PPO 等方法优化 LLM。

## (2) InstructGPT 里训练 RM

训练 RM 前，需要先构造人类偏好数据，通常流程是：先给出一系列 prompt，对同一个 prompt，由 SFT 模型生成多个候选回答，再由人工标注员对这些回答进行排序或偏好比较。

RM 的训练目标是为了对给定 prompt 下的 response 打分，分数反映人类偏好强弱。对于给定的一组 (prompt, response)，输出一个标量分数 $r_\theta(x,y)$，使偏好的回答比未被偏好的回答得分更高。

InstructGPT 中使用的是类似 Bradley-Terry / pairwise preference 的损失。
对于一对回答 $(y_w, y_l)$，RM 损失函数的优化目标为 $L(\theta)=-log \sigma(r_\theta(x,y_w)-r_\theta(x,y_l))$。

其含义是：如果 RM 给人类偏好的回答 $y_w$ 打出比 $y_l$ 更高的分数，那么 $r_\theta(x,y_w)-r_\theta(x,y_l)$ 会变大，$\sigma(\cdot)$ 更接近 1，loss 也会更小。因此，RM 的训练本质上是一个成对偏好学习问题。

# 4. 训练RM时，无论是instruct GPT还是DPO， 为什么loss里有log和sigmod函数？ 直接用reward相减不行吗？



# RM会过拟合吗?过拟合表现是什么?如何评估奖励模型好不好?



# RLHF(指openai instruct GPT论文中)，训练LLM的损失函数是什么？

$LPPO(\Phi) = E_t[min(r_t(\Phi) \hat{A}^t, clip(r_t(\Phi), 1-\varepsilon, 1+\varepsilon) \hat{A}^t)]$  

$r_t(\phi) = \frac{\pi_{\phi}(a_t \mid s_t)}{\pi_{\theta_{\mathrm{old}}}(a_t \mid s_t)}$

# 了解RLHF-PPO吗，里面需要训练几个模型，加载几个模型



# RLHF-PPO里，reward的设计是什么，绝对优势估计是什么



# RLHF-PPO训练的损失函数公式


