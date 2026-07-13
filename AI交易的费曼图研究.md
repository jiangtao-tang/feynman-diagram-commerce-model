# AI交易的费曼图研究

> **副标题**：费曼图模型作为 ERC-8183 交易裁决的数学内核
> **版本**：v1.1 ｜ **日期**：2026-07-12 ｜ **状态**：研究综合稿（聚焦费曼图 × ERC-8183 结合版）

---

## 摘要（Abstract）

本研究源于一个哲学追问：**不考虑任何人类经验，两个 AI Agent 如何判断交易的真实性？** 这一追问将费曼图模型推向更本质的层面——从"人类可信的交易框架"到"宇宙通用的信任数学"。

本文的聚焦主线只有一条：

> **ERC-8183 解决了"两个互不信任的 Agent 怎么放心去交易"的框架问题（任务托管 + 第三方裁决），但它没有解决"这笔交易是否真的可信"的客观判定问题——它的裁决者（Evaluator）的判定仍是主观的、可被恶意操纵的。费曼图模型正好补上这一缺口：以 dNFT 守恒律 Σ(dNFT)=0 作为 ERC-8183 Evaluator 的客观裁决逻辑，把"交易是否可信"从某地址的主观决定推进为数学必然。**

**核心定位**：本模型不为 Agent 经济引入新的商业协议，而是为 ERC-8183 提供**交易级的数学验证内核**。身份认证（ERC-8004）、安全验证（ERC-8126）、支付（x402）已由以太坊基金会推动的 ERC 标准覆盖，本文不再赘述，集中讨论交易裁决层——即费曼图能帮助 ERC-8183 解决什么。

> **状态注**：ERC-8183 截至本文仍为 Draft（在编）；ERC-8126 已于 2026-06 定稿（Final）。本文以"在编标准"的前瞻语气与之对齐。

**一句话点睛**：人类在互联网交易中因认知局限被动选择了"平台"作为信任中介——那是一条弯路；AI 交易没有定域性困境，天然收敛到去中心化的数学验证结构，**本模型主张让 AI 永远不必重走人类"发明平台"的老路**。

---

# 第一部分：费曼图模型的哲学基底

> 本部分回答一个前置问题：**为什么费曼图能胜任"客观裁决"这一角色？** 它是本文技术结合的哲学依据，源自白皮书第 11 章。

## 1.1 哲学追问：剥去人类经验后的信任本质

如果两个 AI Agent 都具备支付和收款能力，且**完全不知道人类的经济学、不知道费曼图、不知道区块链**，它们之间会如何交易？

> **信任的本质是数学，不是经济学。**
> **费曼图是数学模型，不是信任本身。**

**逐层剥离**：

- **第一层（去掉人类经验：文化、法律、声誉）**：剩余物理守恒律（能量守恒、量子不可克隆）、信息论约束（哈希单向性、拜占庭将军问题）——这是宇宙底层协议。
- **第二层（去掉费曼图）**：剩余哈希函数、非对称加密、重复博弈纳什均衡——AI 仍能动交易，但无法形式化信任结构。
- **第三层（去掉哈希函数）**：退化为宇宙最底层交易协议：`能量 ↔ 信息`。

> **核心结论**：费曼图的作用，是给数学信任机制套上**人类可理解的解释框架**。没有牛顿力学苹果也会掉；有了牛顿力学，人类可以预测苹果什么时候掉。

## 1.2 AI Agent 的自发交易机制

两个 AI 通过博弈自发收敛到：

1. **信息层**：直接交换哈希，"我有数据 X，哈希=H(X)"。
2. **博弈层**：发现抵押机制——诚实的长期收益 > 欺诈的一次性收益（重复博弈纳什均衡）。
3. **数学层**：A 给出 X 链上记 `-X`，B 收到 X 链上记 `+X`，数学上必须 `Σ(所有交易) = 0`——这正是 **dNFT 对的本质**。

| 费曼图元素 | AI 交易中的对应 | 是否"知道"这个名字 |
|-----------|----------------|---------------|
| 外线（粒子） | AI Agent 钱包地址 | ❌ 不知道 |
| 顶点（相互作用） | 交易合约执行点 | ❌ 不知道 |
| 传播子（传递） | 物流/IoT 数据传递 | ❌ 不知道 |
| 守恒律 | Σ(dNFT) = 0 | ✅ **数学必然** |
| 创生/湮没 | dNFT 对的生命周期 | ❌ 不知道 |

> **关键洞察**：费曼图不是"设计出来的"，而是**数学结构的必然呈现**——就像 `1+1=2` 不依赖人类而存在。

## 1.3 AI 判断交易真实性的 3 个机制

1. **密码学证明**：Merkle Root 验证物理标识哈希、ZKP 证明交付正确、数字签名。
2. **dNFT 守恒律验证（数学必然）**：Σ(dNFT)=0、资产价值守恒、Real↔Virtual 双世界耦合。
3. **链上状态确认**：交易合约执行后 Virtual 层状态不可篡改，**不需要信任对方，只需信任代码和数学**。

## 1.4 费曼图作为 AI 经济的"商业语法"

本模型为 AI↔AI 经济设计通用商业语法：主语/宾语→外线，动词→顶点，状语→传播子，语法规则→守恒律，语义验证→智能合约执行。

> **当 AI 之间需要交易时，它们不会发明"平台"，而是直接收敛到费曼图结构。**

### 1.4.1 平台：人类认知局限的补丁，而非商业的必然

上句的深刻之处在于：**"平台"不是商业的最优形态，而是人类认知约束下的次优妥协。** 理解这一点，才能看清费曼图模型不是"又一个去中心化方案"，而是 AI 交易唯一自然的组织形态。

人类的互联网交易史是一条被动的弯路。其因果链清晰可见：

- 人类需要交易，就需要信任；
- 人类受困于自身的**定域性困境**——无法低成本验证陌生人、无法数学化地确认一笔交易是否真实；
- 在找不到去中心化信任解的前提下，被迫选择一个中心化的"平台"来充当信任中介；
- 平台一旦成为信任锚点，便自然滑向垄断，最终反过来锁定整个生态（商家被抽佣、用户被锁定、数据被圈养）。

每一步都"没得选"。这不是人类理性失败，而是人类**受制于自身生物学/社会性局限**的必然结果——"平台"本质上是为这个局限打的一块补丁。

AI 的经济组织则不存在这条弯路。其收敛路径截然不同：

- AI 之间交易，并不需要"信不信你这个人"；
- 它只需验证"这笔交易的结构是否守恒"；
- Σ(dNFT)=0 是数学必然，不依赖任何中间人意志；
- 因此 Agent 直接彼此收敛到费曼图结构，**根本不会产生"平台"这个需求**。

对人类而言必需的东西，对 AI 而言是一个**没有存在理由的冗余层**。人类经济是树状的（平台 → 商家 → 用户，信任自上而下传递）；AI 经济是网状的（Agent 通过数学结构直接互认，信任在任意两点间点对点成立）。

> **核心命题：平台是人类走不通去中心化时的退路。AI 没有这个退路困境，也就没有"平台时代"——它直接跳到基于数学验证的去中心化结构。费曼图模型所主张的，不是"让 AI 也用上平台"，而是"让 AI 永远不必发明平台"。这是 AI 经济与人类互联网经济在组织形式上的结构性分野，也是我们在设计本协议栈时反复提醒自己的一条红线：千万不要再走人类走过的弯路。**

## 1.5 哲学小结

信任是宇宙的数学性质，不是人类社会的发明。**费曼图是宇宙交易的基本语法**——这一性质，正是它能作为"客观裁决"而非"主观裁决"的深层理由：数学必然不依赖于任何裁决者的意志。

---

# 第二部分：ERC-8183 的框架与留下的裁决缺口

> 本部分先交代 ERC-8183 是什么、解决了什么，再点出它留下的核心缺口——这正是费曼图的落点。身份认证（ERC-8004）、安全验证（ERC-8126）、支付（x402）已由 ERC 标准覆盖，此处不再展开。

## 2.1 ERC-8183 是什么：怎么放心去交易

ERC-8183（Agentic Commerce，Draft）定义了一个**原子级 Job 托管协议**，专门解决"两个互不信任的 Agent 怎么放心交易"。核心是一个 Job（任务）原语：

| 角色 | 职责 |
|------|------|
| **Client**（客户/雇佣方） | 创建 Job、设预算、注资 |
| **Provider**（服务方/交付方） | 干活、submit 交付物 |
| **Evaluator**（评估者） | **唯一可标记完成/拒绝的裁决方** |

四状态机：**Open → Funded → Submitted → Terminal**
- Funded：Client 把钱锁进合约托管，Provider 可以开始干
- Submitted：Provider 交付产物（可带 hash 引用）
- Terminal 三结局：**Completed**（放款给 Provider）/ **Rejected**（退给 Client）/ **Expired**（过期自动退 Client）

## 2.2 ERC-8183 解决了什么

- **锁资 + 第三方裁决**，同时消除了"先付赌交付"和"先干赌付款"两种风险。
- **Evaluator 是信任破局点**：无它，交易退化为双向赌博。
- Hook 模块化：声誉/SLA/自定义支付走 Hook；但 `claimRefund` **故意不可 Hook**，防止恶意 Hook 永久锁资（安全 > 灵活）。
- 单一 ERC-20 代币，降低攻击面。

一句话：**ERC-8183 把"怎么放心去交易"的框架问题解决了——有了托管、有了裁决者、有了状态机。**

## 2.3 ERC-8183 留下的核心缺口：Evaluator 如何客观裁决

ERC-8183 定义了"**谁裁决**"（Evaluator 这个角色），但**没有定义"如何客观裁决"**。它的安全条款自己明确警告：

> **"malicious evaluator can arbitrarily complete or reject"**
> （恶意 Evaluator 可以任意放行或拒绝）

这意味着：**Evaluator 的判定，本质是某个地址的主观决定**。标准仅建议"用声誉/staking 防范"，但：
- 声誉是经验性的，可被刷；
- staking 只提高作恶成本，不消除作恶能力；
- 高价值 Job 的裁决仍由人/合约主观判断"交付是否合格"。

这就是 8183 留出的、费曼图要填补的缺口。

## 2.4 其他层已被解决（本文不展开）

- **身份认证**：ERC-8004（Agent 身份/声誉/验证注册表）已覆盖。
- **安全验证**：ERC-8126（Final，Agent 五层安全验证 + 风险评分）已覆盖"这个 Agent 是否安全"。
- **支付**：x402（HTTP 稳定币微支付）已覆盖"怎么付钱"。

本文**不再赘述这三层**，只聚焦它们之下的交易裁决层。

---

# 第三部分：费曼图帮 ERC-8183 解决什么（全文核心）

> 本节是本文的重心。费曼图模型对 ERC-8183 的价值，不在于另起一套商业协议，也不在于为它"打补丁"，而在于**把 8183 的信任范式从"人类经验裁决"替换为"客观数学必然裁决"**。3.2 节将先批驳 8183 固有的信任范式错配——这正是费曼图贡献的出发点。

## 3.1 核心命题：费曼图 = ERC-8183 Evaluator 的数学内核

ERC-8183 允许 Evaluator **本身就是一个智能合约**（验证 ZKP 或聚合信号）。费曼图模型的 Feynman Evaluator 正是这样一种 Evaluator 实现：

- ERC-8183 提供**裁决框架**（谁可以 complete/reject、何时触发）；
- 费曼图提供**裁决标准**（Σ(dNFT)=0 是否成立）；
- 二者结合 = **客观、自动、去主观的交易裁决**。

> 原白皮书 11.8 的 25 行伪代码，正是这一命题的最初表达：`delta = DNFTRegistry.getDelta(jobId)` → `conservationHolds(delta)` → complete/reject。

## 3.2 ERC-8183 的人类经验残留：信任范式的根本错配

上一节把费曼图定位为 8183 的"Evaluator 数学内核"。但要精确地说，费曼图对 8183 的贡献**不是一个补丁，而是一次信任范式的替换**——因为 ERC-8183 在"可信标准层"并未脱离人类商业信任模式，而是把人类解决信任的经典手段原样搬上链并"可编程化"。这也是为什么第一部分哲学基底坚持"AI 只认数学、不认主体"在此成为判准：8183 的信任逻辑恰恰与人类经验绑死。

### 3.2.1 划界：批驳的是"可信标准"，不是"托管机制"

必须严格区分，避免无的放矢：

- **不批驳**：ERC-8183 的托管与状态机（Open→Funded→Submitted→Terminal）是中性的技术机制，本身不承载信任判断。
- **批驳**：建立在托管之上的**信任裁决层**（规范称之为 "trust layer"）——即"一笔交易 / 一个 Agent 是否可信由什么决定"。

一个可信标准只要依赖以下任一特征，即属"带人类经验判断"：依赖某主体的主观意志（法官/仲裁）、依赖经验性社会建构（声誉/口碑）、依赖专家经验评分、依赖主观质量预期（"好不好"）、依赖金额/规模的经验分层（大额才抵押）。反之，仅依赖数学/物理结构约束（守恒律、哈希单向性）的可信标准，与 AI 逻辑自洽。

### 3.2.2 五大人类经验残留点（带规范原文证据）

| # | 8183 的人类经验设计 | 规范 / 解读证据 | 与 AI 逻辑冲突 |
|---|-------------------|---------------|---------------|
| **R1** | **法官式 Evaluator 主观裁决**：信任 = "某地址说了算" | Abstract："an evaluator who alone may mark the job completed"；Security："a malicious evaluator can complete or reject arbitrarily" | 把信任委托给"裁决主体"，是拟人化信任；AI 只需结构验证，不需裁决者 |
| **R2** | **主观质量判断**："报告好不好 / 设计接不接受"作为放款前提 | DecipherClub 自承："for subjective tasks... evaluation requires judgment that current on-chain mechanisms cannot reliably provide" | 把人类审美/需求标准内置为闭环必要环节，等于承认脱离人类无法闭环 |
| **R3** | **声誉作信任基础**：ReputationHook / ReputationGateHook 以 ERC-8004 声誉为接活门槛 | TryEthernal："the evaluating agent's own reputation becomes the trust basis" | 声誉 = 口碑社会链上复刻（老字号效应），可被刷；AI 只需验证数学守恒通过率 |
| **R4** | **经验风险评分前置闸**：ERC-8126 的 0–100 经验分作为 Evaluator 前置闸 | 8126 评分为经验公式，非第一性原理推导 | 信任委托给专家权威；交易可信性与主体评分无关 |
| **R5** | **金额阈值风控**："高价值才要声誉/staking" | Security："Use reputation or staking for high-value jobs" | 人类风险管理经验；费曼图守恒律普适，与金额无关 |

> **补充留白（规范自身承认）**：①"选谁当 Evaluator"仍是 open design space——把最关键的人类判断（选谁裁决）甩给实现层；②无内置仲裁，默认人类司法（UMA/Kleros）补位——信任基础仍锚定人类社会。

### 3.2.3 平衡说明：8183 的"数学化空间"及其局限

为学术严谨必须承认，ERC-8183 **并非全人类经验**：规范明确允许 Evaluator 是**智能合约**（如 ZK verifier），对**确定性任务**（计算、证明、数据转换）可做客观裁决——这部分确实非人类经验，与费曼图方向一致。

但局限有三：①**默认 / 兜底仍是主观**——生态 Hook（ReputationHook 等）默认走声誉 + 主观 Agent 评估路径；②**主观任务无法数学化**——写作、设计、分析占 Agent 经济很大比例，规范自己承认这些 "requires judgment that current on-chain mechanisms cannot reliably provide"；③**信任基础未替换**——即便用合约 Evaluator，规范仍把信任锚定在"选对 evaluator（哪怕是合约）"而非"结构必然"。

**结论**：ERC-8183 是一个**混合标准**——既给数学化接口，又以人类经验为默认信任基础。它称不上"纯 Agent 原生的可信标准"，而是"人类信任模式的可编程封装"。

### 3.2.4 费曼图对照：每个残留点 → AI 逻辑的正确形态

| ERC-8183 的人类经验点 | 费曼图的 AI 逻辑替代 | 为何符合 AI 逻辑 |
|---------------------|---------------------|----------------|
| R1 法官式 Evaluator 主观裁决 | Σ(dNFT)=0 客观必然：通过则 complete，不通过则 reject | 裁决来自**结构**，不来自**主体意志** |
| R2 主观质量判断 | "质量"翻译为可验证结构（L2 配对完整性 / L3 价值守恒 / L5 双世界耦合） | 可信性 = **结构满足**，非"人类满意" |
| R3 声誉作信任基础 | 链上"数学守恒通过率"（结构事实，无口碑累积） | 信任来自**可重复验证的数学**，非经验偏见 |
| R4 经验风险评分前置闸 | 不依赖主体安全性，只验证**本次交易** Σ(dNFT)=0 | 交易级可信独立于主体评分 |
| R5 金额阈值风控 | 守恒律普适，任何金额同等待验证 | 信任需求与金额无关，是结构必然 |

> **一句话概括差异**：
> ERC-8183 让 Agent **"像人一样做买卖"**（有法官、有口碑、有质检、有风控）；
> 费曼图让 Agent **"用数学证明买卖真实"**（无法官、无口碑、无质检、无金额分层）。

---

## 3.3 四大痛点 → 四个解法（费曼图对 8183 的具体贡献）

| # | ERC-8183 的痛点 | 标准现状 | 费曼图模型的解法 |
|---|----------------|---------|-----------------|
| **P1** | **信任范式错误：裁决主体是人类经验的代理人**（"法官式 Evaluator" 把信任寄托于某地址的主观意志，malicious evaluator 可任意 complete/reject） | 仅建议"用声誉/staking 防"，但其信任基础本身即人类商业的法官/声誉/质检模式——**非安全漏洞，而是范式根错**（论证见 3.2） | 以 **dNFT 守恒 Σ(dNFT)=0** 作为唯一裁决判据——通过则 complete、不通过则 reject，**裁决由数学决定，不由地址决定**；费曼图不是"修补 8183 的漏洞"，而是**替换其整个信任范式** |
| **P2** | **高价值 Job 的 Evaluator 没有经济背书** | 标准建议 staking 但未定义具体机制 | **保险池/担保池 = Evaluator 的 staking 实现**（本文第五部分）：保证金池覆盖恶意/失误裁决造成的损失，且担保者退出受动态风险参数 E 约束 |
| **P3** | **交付验证依赖人工/主观 Evaluator 判断"交付是否合格"** | Evaluator 需主观判断交付物 | **dNFT 守恒 + 五层校验（L1–L5）自动验证交易结构**：deliverable 不是"文件"而是"交易完成的数学声明"，由合约客观校验，无需人类判断 |
| **P4** | **跨 Agent 信任难以建立、无统一标准** | 各 Job 独立 Evaluator，信任不可移植 | **Σ(dNFT)=0 是宇宙通用的数学必然**，任何 Agent 都能独立验证，信任从"依赖某 Evaluator 声誉"变为"依赖数学结构"——真正去信任化 |

**一句话总结四大贡献**：费曼图把 ERC-8183 的裁决从"某人/某合约说了算"变成"数学说了算"，并补上经济兜底与自动验证。

## 3.4 dNFT 守恒律作为 Evaluator 的客观判据

物理定义（单笔交易的闭合条件，非全局不变量）：

```
创生前：          Σ(dNFT) = 0
创生后：          Σ(dNFT) = (+1) + (-1) = 0       // 商家铸造 dNFT 对
交易合约后：      Σ(dNFT) = (+1_买家) + (-1_物流) = 0  // V₁ 顶点
湮没后：          Σ(dNFT) = 0                        // V₂ 顶点，对消失
```

等价表述（dNFT 状态机：Created→Locked→InTransit→Delivered→Annihilated）：

| 守恒维度 | 物理含义 | 作为 Evaluator 判据 |
|---------|---------|-------------------|
| 数量守恒 | Σ(chirality) = 0 | 活跃 dNFT 代数和为 0 |
| 配对守恒 | 每个 +dNFT 有唯一 -dNFT | 所有 +dNFT.pairId 在 -dNFT 中匹配 |
| 价值守恒 | Σ(value×chirality) ≈ 0 | 正负价值代数和 ≤ ε |
| 状态守恒 | 交易结束所有 dNFT 湮没 | `all(status == Annihilated)` |
| 双世界守恒 | ΔReal + ΔVirtual = 0 | Real 层物理事件锚定 Virtual 层状态变更 |

> 这正是 ERC-8183 Evaluator 一直缺失的"客观放行逻辑"——8183 只说"Evaluator 决定放行"，费曼图说"放行当且仅当 Σ(dNFT)=0"。

## 3.5 组件映射：我们的组件 → ERC-8183 角色

| 我们的组件 | 在 ERC-8183 中的角色 | 解决的痛点 |
|-----------|---------------------|-----------|
| dNFT | Job 的"交付物/价值载体"；状态变更 = submit 内容 | P3 |
| Σ(dNFT)=0 | **Evaluator 的客观放行判据** | P1 |
| Real/Virtual 双世界 | 一笔 Agent 交易的两个侧面（链上资产 ↔ 链下服务） | P1（防造假交付） |
| 保险池/担保池 | **Evaluator 的 staking**（高价值 Job 防作恶） | P2 |
| 信用评分 | 回写 ERC-8004 声誉的"数学可信度"维度 | P4（长期信任） |
| 链上验证轨迹 | attestation hash（complete/reject 理由哈希） | P1/P4（可审计） |

---

# 第四部分：Feynman Evaluator Contract——把数学变成 8183 的 Evaluator

> 本部分将 π 白皮书 11.8 草图展开为可实现设计，核心：如何把 Σ(dNFT)=0 翻译成 Solidity 链上可执行的校验逻辑，作为 ERC-8183 Evaluator 的实现。

## 4.1 实现缺口

原草图仅含 `delta = DNFTRegistry.getDelta(jobId)` → `conservationHolds(delta)` → complete/reject。四个缺失：

| # | 缺失项 | 说明 |
|---|--------|------|
| A | **DNFTRegistry 合约** | getDelta() 的数据源 |
| B | **conservationHolds() 算法** | Σ(dNFT)=0 在合约里怎么检查 |
| C | **Job ↔ Order 映射** | ERC-8183 jobId 与 Escrow orderId 对接 |
| D | **deliverable 格式** | Provider 提交什么给 Evaluator 验 |

## 4.2 组件 A：DNFTRegistry（数据源）

```solidity
contract DNFTRegistry {
    struct DNFTDelta {
        uint256 tokenId;
        int8 chirality;          // +1 或 -1
        address fromOwner;
        address toOwner;
        DNFTStatus oldStatus;
        DNFTStatus newStatus;
        uint256 value;
        bytes32 eventHash;       // 触发变更的链上事件哈希（顶点签名）
        VertexType vertexType;   // CREATE / TRADE / ANNIHILATE / RETURN
    }
    struct JobSnapshot {
        uint256 jobId;
        uint256[] tokenIds;
        mapping(uint256 => DNFTDelta[]) deltasByToken;
        bool isClosed;
        bytes32 closureProof;
    }
    function recordDelta(uint256 _jobId, uint256 _tokenId,
        VertexType _vertexType, bytes32 _eventHash) external onlyAuthorized { ... }
    function getDelta(uint256 _jobId) external view returns (DNFTDelta[] memory) { ... }
}
```

## 4.3 组件 B：conservationHolds()——五层校验（核心算法）

单一 `sum(chirality)==0` 不够（攻击者可造假配对、价值不匹配、跳步）。需 **5 层递进校验**：

```solidity
function conservationHolds(DNFTDelta[] memory delta) internal pure
    returns (bool conserved, string memory reason, ConservationProof memory proof)
{
    // L1: 数量守恒（仅统计未湮没的活跃 dNFT）
    int256 netChirality = 0;
    for (uint i = 0; i < delta.length; i++)
        if (delta[i].newStatus != Annihilated)
            netChirality += delta[i].chirality;
    if (netChirality != 0) return (false, "L1_FAIL", emptyProof());

    // L2: 配对完整性（每个 pairId 恰好 +1/-1）
    if (!checkPairingIntegrity(delta)) return (false, "L2_FAIL", proof);

    // L3: 价值守恒 |Σ(value×chirality)| ≤ ε（百万分之一容差）
    (uint256 posVal, uint256 negVal) = getBilateralValues(delta);
    if (abs(int256(posVal) - int256(negVal)) > int256(posVal / 1_000_000))
        return (false, "L3_FAIL", proof);

    // L4: 顶点序列合法（CREATE→TRADE→...→ANNIHILATE，不可跳步）
    if (!checkVertexSequence(delta)) return (false, "L4_FAIL", proof);

    // L5: Real-Virtual 双世界耦合（每个 Virtual 变更都有 Real 锚点签名）
    if (!checkRealVirtualCoupling(delta)) return (false, "L5_FAIL", proof);

    return (true, "ALL_PASSED", proof);
}
```

**各层要点**：
- **L1** 统计"当前活跃"而非"历史创建"；空 delta 通过（纯服务型任务）。
- **L2** 一笔交易一对 dNFT（多 SKU = 多独立 pairId）。
- **L3** 容差 ε=0.0001% 处理舍入/汇率差；保险费不计入价值不平衡。
- **L4** 防"幽灵 dNFT"（无正规创生直接流通）。
- **L5**（最难也最有创新）：每个 Virtual 变更必须有对应 Real 层物理事件签名（IoT/NFC、ECDSA、预言机锚点）。FeynmanEvaluator **信任签名存在性，不验证内容真实性**（类比 TLS 证书链）。这是区分费曼图 Escrow 与普通托管的关键。**⚠ 纯数字商品豁免 L5**：数字商品的交付本身发生在链上或链上可验证（哈希匹配、智能合约执行结果），不存在"Real 世界物理事件需锚定"的问题，L5 仅对含实物 RWA 的交易生效。

## 4.4 组件 C：Job ↔ Order 映射（JobBridge）

| 系统 | 主键 | 状态机 |
|------|------|--------|
| ERC-8183 Job | jobId (uint256) | Open→Funded→Submitted→Terminal |
| Escrow Order | orderId (bytes32) | Created→FUNDED→SHIPPED→DELIVERED→COMPLETED |

JobBridge 做双向映射；**触发模式 A（推荐）**：ERC-8183 Hook 在 `submit()` 后自动挂载 `FeynmanEvaluator.evaluate()`。

## 4.5 组件 D：deliverable 格式

submit() 在费曼图语境下不是"文件"，而是**交易完成的数学声明**：

```solidity
struct FeynmanDeliverable {
    uint256 orderId;
    uint256 dnftPairId;
    DNFTStatus expectedFinalState;   // 应为 Annihilated
    bytes32 realWorldEvidenceCID;    // Real 层证据 CID
    bytes32[] oracleSignatures;      // 物流节点预言机签名
    bytes32 merkleRoot;              // 链下证据 Merkle root（用于 ZKP）
}
```

## 4.6 完整 evaluate() 流程

```
FeynmanEvaluator.evaluate(jobId, deliverable):
  Step 1 解码 & 映射: decode → FeynmanDeliverable; JobBridge→orderId; 校验 job==Submitted
  Step 2 获取: DNFTRegistry.getDelta(orderId); DynamicNFT 实时状态
  Step 3 交叉验证: pairId/状态/oracle 覆盖度 → 不通过 reject("deliverable mismatch")
  Step 4 conservationHolds(): L1→L5，任意层失败 → reject(proof+layer+reason)
  Step 5 证明 & 执行: proof=keccak256(delta+result+blockhash);
       DNFTRegistry.markClosed(orderId, proof);
       ERC8183Job.complete(jobId, proof)  ← 放款给 Provider
```

## 4.7 Gas 优化（工程关键）

全量跑完 5 层预估 200k–600k+ gas。推荐**增量验证**：在 placeOrder/confirmDelivery/confirmReceipt 每个顶点事件时做增量检查，evaluate() 最终只做 ~50k 轻量一致性确认。另可选链下计算 + ZKP（verify 固定 ~200k，与 delta 数无关）。

## 4.8 安全与攻击面

| 攻击向量 | 影响层 | 防御 |
|---------|--------|------|
| 虚假 dNFT 创生 | L1/L2 | DNFTRegistry 仅接受 DynamicNFT 授权写入 |
| 价值操纵 | L3 | 价值创生时预言机报价锁定不可变 |
| 跳步攻击 | L4 | 顶点序列强制检查 |
| 预言机作恶 | L5 | ≥3 独立源 + staking 惩罚 |
| Evaluator 合约被替换 | 全局 | Evaluator 地址 Job 创建时锁定；Proxy 升级需 Timelock |

**最重要缓解 = 第五部分保险池/staking**：即使 Evaluator 数学逻辑被突破，经济层仍有担保金覆盖。

## 4.9 实现路线图

```
Phase 0（基础设施）: DynamicNFT / DNFTRegistry / JobBridge / ERC-8183 Draft 实现
Phase 1（MVP）: FeynmanEvaluator v0.1 仅 L1+L2；手动触发
Phase 2（生产就绪）: 加 L3+L4；deliverable v1.0；Hook 自动触发
Phase 3（完整）: 加 L5（预言机）；ZKP 可选；保险池 staking 集成；形式化验证
Phase 4（生态）: 多链部署；ERC-8004 声誉回写；Kleros 仲裁 fallback
```

## 4.10 开放问题

> **已确定的设计决策（从开放问题移出）**：
> - ~~纯数字商品 L5 豁免~~ → **已确定：** 纯数字商品（无实物 RWA）豁免 L5，因其交付本身链上可验证，L5 仅对含实物 RWA 的交易生效。详见 4.6 L5 说明。

| # | 问题 | 建议 |
|---|------|------|
| 1 | 部分退货场景守恒怎么算？ | dNFT 分数化 / 子配对 |
| 2 | 多商家联合订单守恒范围 per-job 还是 per-merchant？ | Per-job |
| 3 | 跨链场景 δ 如何收集？ | CCIP/LayerZero 同步，或限制单链闭环 |
| 4 | conservationHolds 跑在 EVM 还是链下？ | Phase 1-3 链上；Phase 4 可选 ZKP 链下 |

---

# 第五部分：经济背书——保险池作为 ERC-8183 Evaluator staking

> 本部分对应 8183 痛点 **P2**：高价值 Job 的 Evaluator 需要经济背书。保险池/担保池正是 ERC-8183 所建议的 staking 的工程实现，为客观裁决再上一道经济兜底。

## 5.1 两层架构

- **Layer 1（主层）：商家保证金池**——按商家（非按交易）建立，覆盖常规赔付。
- **Layer 2（超额层）：第三方 DeFi 超额赔付保险**（Nexus Mutual 等）——当 Layer 1 不足时触发。

## 5.2 保险比例分层（基数 = 商品价格 + 物流费用）

| 商品+物流总价 | 比例 | 说明 |
|---|---|---|
| <$50 | 20-25% | 低价，运费占比高 |
| $50-$500 | 15% | 主流区间 |
| $500-$2000 | 10% | 中高价 |
| >$2000 | 5% | 高价 |
| 奢侈品/数字商品 | 最高 30% | 高风险类目上限 |

保险费由**卖家支付**（交易金额 × 3%），买家无额外费用。

## 5.3 动态风险参数与担保者进出

加权风险敞口：`E = Σ(B_i × r_i)`，B_i=在途交易金额，r_i=对应担保比例。

担保者退出条件：退出后 `C' = C − D ≥ E` 才允许；否则只允许部分退出 `D' = C − E`。新交易风险提示（不阻止成交）：若 `C < E + V_new × r_new`，仅做 ⚠️ 提示，购买自主决定。

## 5.4 商家最低出资（动态）

`S_min = (A + B) × r × 10%`，A=dNFT 在合约中总额，B=在途 dNFT 总额，r=价格区间担保比例。信用分提升 → r 下降 → S_min 下降。

## 5.5 收益分配（动态，无固定比例）

某参与者应得 = 本周保费 ×（出资额 × 质押天数）/ 池内总权重。按周结算，可随时提取。

## 5.6 与 ERC-8183 的对接（作为 staking）

| 保险池概念 | ERC-8183 对应 |
|---|---|
| 保证金池 / 担保者质押 | **Evaluator staking**（防高价值 Job 作恶，对应痛点 P2） |
| 动态风险参数 E | Evaluator 放行前的资本充足性闸门 |
| 违约扣押 / 赔付 | Evaluator complete/reject 的经济后果背书 |
| Kleros 仲裁 | Evaluator 裁决的证据层 |

---

# 第六部分：开放问题与研究议程（综合）

1. **协议层**：ERC-8183 定稿后接口锁定；验证商共谋的多 Evaluator 冗余方案；8126 经验评分 → 守恒律第一性原理评分的映射（但 8126 非本文重点）。
2. **实现层**：L5 双世界耦合的预言机网络成熟度（~~纯数字商品 L5 豁免规则→已确定，见 4.10~~）；ZKP 链下验证与 ERC-8126 PDV 的可组合证明；跨链 δ 同步。
3. **经济层**：保险池 staking 与 Evaluator 放行的资本充足性闸门联动；信用分动态下调 r 的长期博弈稳定性。

---

# 附录 A：ERC 标准状态（截至 2026-07-12）

| 标准 | 状态 | 角色 | 本文处理 |
|------|------|------|---------|
| ERC-8004 | Draft | Agent 身份/声誉 | 一笔带过（已被覆盖） |
| ERC-8126 | **Final（2026-06 定稿）** | Agent 五层安全验证 + 风险评分 | 一笔带过（已被覆盖） |
| **ERC-8183** | **Draft** | **Agentic Commerce（Job 托管 + Evaluator）** | **本文核心结合对象** |
| x402 | 实践方案 | HTTP 稳定币微支付 | 一笔带过（已被覆盖） |

> 注：本文以"在编标准"前瞻语气与 ERC-8183 对齐；接口以最终定稿版为准。费曼图模型不替代上述任一标准，仅补其交易裁决层缺口。

# 附录 B：术语对照

| 术语 | 含义 |
|------|------|
| dNFT | 动态非同质化代币，承载交易价值与状态 |
| Σ(dNFT)=0 | dNFT 守恒律：单笔交易闭合条件，作为 ERC-8183 Evaluator 客观判据 |
| Vertex（顶点） | 交易合约执行点（V₁ 交易、V₂ 湮没） |
| Propagator（传播子） | 物流/IoT 数据传递 |
| Feynman Evaluator | 实现 ERC-8183 Evaluator 接口的合约，以守恒律客观裁决（解决 P1） |
| Evaluator staking | 高价值 Job 防作恶的经济背书（对应保险池，解决 P2） |

---

*文档版本：v1.1 ｜ 日期：2026-07-12 ｜ 状态：研究综合稿（聚焦费曼图 × ERC-8183 结合版）。基于《去中心化商业费曼图模型白皮书 v1.2 第十一章》+《费曼图模型与 ERC 协议栈对齐方案》+《FeynmanEvaluator 实现分析》综合重写而成。*
