# Feynman Evaluator Contract 实现分析（ERC-8183 Evaluator 接口的数学内核）

> 本文档深入拆解白皮书 11.8 节的 FeynmanEvaluator 合约：从当前伪代码到可实现的完整设计。
> 核心问题：如何把 dNFT 守恒律 Σ(dNFT)=0 这个数学约束，翻译成 Solidity 链上可执行的校验逻辑。

---

## 一、问题定位：11.8 当前状态 vs 实现缺口

### 1.1 白皮书现有内容（仅伪代码）

```
interface IFeynmanEvaluator {
    function evaluate(uint256 jobId, bytes calldata deliverable)
        external returns (bool passed, bytes32 proof);
}
// evaluate 内部：
//   1. delta = DNFTRegistry.getDelta(jobId)     ← 这是什么？
//   2. conserved = conservationHolds(delta)      ← 怎么算？
//   3. proof = keccak256(abi.encode(delta, ts))
//   4. 通过 → complete / 不通过 → reject
```

### 1.2 四个关键缺失

| # | 缺失项 | 白皮书中的位置 | 说明 |
|---|--------|---------------|------|
| **A** | **DNFTRegistry 合约不存在** | 第 3826 行只列了文件名 `contracts/DNFTRegistry.sol`，无代码 | getDelta(jobId) 的数据源 |
| **B** | **conservationHolds() 算法未定义** | 第 3283 行一行调用 | Σ(dNFT)=0 在合约里怎么检查？ |
| **C** | **与 ERC-8183 Job 的映射关系不明** | jobId 是 ERC-8183 的还是 Escrow 的？ | 两套 ID 系统如何对接 |
| **D** | **deliverable 参数格式未定义** | bytes calldata deliverable | Provider 提交了什么？Evaluator 拿什么验？ |

这四个缺失就是本文要逐一填补的实现细节。

---

## 二、核心概念翻译：从物理学到 Solidity

### 2.1 "Σ(dNFT)=0" 到底在验什么？

回到白皮书的物理定义（第 1437–1444 行）：

```
创生前：          Σ(dNFT) = 0
创生后：          Σ(dNFT) = (+1) + (-1) = 0       // 商家铸造 dNFT 对
交易合约后：      Σ(dNFT) = (+1_买家) + (-1_物流) = 0  // V₁ 顶点
湮没后：          Σ(dNFT) = 0                        // V₂ 顶点，对消失
```

**关键洞察**：Σ(dNFT)=0 不是全局不变量（系统内可以有多笔交易同时存在），而是**单笔交易的闭合条件**——一笔交易涉及的 dNFT 代数和在交易结束时必须归零。

翻译成合约语言：

> **conservationHolds(jobId)** = 检查该 Job 涉及的所有 dNFT 状态变更是否构成一个"闭合费曼图"——即所有 +dNFT 都找到了对应的 -dNFT 配对并完成湮没。

### 2.2 dNFT 状态机（已有的定义）

白皮书第 1670–1700 行和 DynamicNFT.sol（附录代码第 3886 行起）已定义：

```
dNFT 状态流转：
  Created → Locked → InTransit → Delivered → Annihilated（湮没）
                    ↘ Refunded

每个 dNFT 有：
  tokenId: uint256          // 唯一标识
  pairId: uint256           // 配对 ID（+dNFT 和 -dNFT 共享同一个 pairId）
  chirality: int8           // 手性：+1 或 -1
  owner: address            // 当前持有者
  status: enum Status       // 状态
  value: uint256            // 锚定价值（商品价格）
  pairedTokenId: uint256    // 配对方 Token ID
```

**守恒律等价表述**：

| 守恒维度 | 物理含义 | Solidity 检查方式 |
|---------|---------|------------------|
| 数量守恒 | Σ(chirality) = 0 | `sum(d.chirality for d in job_dNFTs) == 0` |
| 配对守恒 | 每个 +dNFT 有唯一 -dNFT 配对 | 所有 +dNFT.pairId 在 -dNFT 中都能找到匹配 |
| 价值守恒 | Σ(value × chirality) ≈ 0 | 正负价值代数和接近零（允许舍入误差 ≤ 1e-12） |
| 状态守恒 | 交易结束时所有 dNFT 已湮没 | `all(d.status == Annihilated)` |
| 双世界守恒 | ΔReal + ΔVirtual = 0 | Real 层资产转移量 = Virtual 层信用变化量 |

---

## 三、组件 A：DNFTRegistry 设计（数据源）

### 3.1 职责

DNFTRegistry 是 FeynmanEvaluator 的**只读数据源**。它不持有 dNFT（那是 DynamicNFT 合约的事），而是记录**每个 Job/Order 涉及哪些 dNFT、以及这些 dNFT 的状态变更轨迹**。

### 3.2 核心数据结构

```solidity
contract DNFTRegistry {
    DynamicNFT public dnftContract;

    // 单次 dNFT 状态变更记录
    struct DNFTDelta {
        uint256 tokenId;
        int8 chirality;          // +1 或 -1
        address fromOwner;       // 变更前持有者
        address toOwner;         // 变更后持有者
        DNFTStatus oldStatus;
        DNFTStatus newStatus;
        uint256 value;           // 锚定价值
        uint256 timestamp;
        bytes32 eventHash;       // 触发该变更的链上事件哈希（顶点签名）
        VertexType vertexType;   // CREATE / TRADE / ANNIHILATE / RETURN
    }

    // 一个 Job 的完整 dNFT 变更快照
    struct JobSnapshot {
        uint256 jobId;
        uint256[] tokenIds;              // 该 job 涉及的所有 dNFT
        mapping(uint256 => DNFTDelta[]) deltasByToken;  // 每 token 的变更历史
        uint256 totalPositiveValue;      // +dNFT 总价值
        uint256 totalNegativeValue;      // -dNFT 总价值
        bool isClosed;                   // job 是否已闭合
        bytes32 closureProof;            // 闭合证明哈希
    }

    enum VertexType { CREATE, TRADE, ANNIHILATE, RETURN }
}
```

### 3.3 关键函数

```solidity
    // 由 Escrow 合约在关键节点调用，记录 dNFT 变更
    function recordDelta(
        uint256 _jobId,
        uint256 _tokenId,
        VertexType _vertexType,
        bytes32 _eventHash
    ) external onlyAuthorized {
        // 从 DynamicNFT 合约读取当前状态
        (int8 chirality, , address owner, DNFTStatus status, uint256 value, , ) 
            = dnftContract.getDNFTInfo(_tokenId);
        
        // 推算 from/to（从上一个 delta 推导）
        DNFTDelta[] storage history = jobDeltas[_jobId][_tokenId];
        address fromOwner = history.length > 0 ? 
            history[history.length - 1].toOwner : address(0);
        
        DNFTStatus oldStatus = history.length > 0 ?
            history[history.length - 1].newStatus : DNFTStatus.Created;
        
        history.push(DNFTDelta({
            tokenId: _tokenId,
            chirality: chirality,
            fromOwner: fromOwner,
            toOwner: owner,
            oldStatus: oldStatus,
            newStatus: status,
            value: value,
            timestamp: block.timestamp,
            eventHash: _eventHash,
            vertexType: _vertexType
        }));

        emit DeltaRecorded(_jobId, _tokenId, _vertexType, chirality);
    }

    // FeynmanEvaluator 调用的核心接口
    function getDelta(uint256 _jobId) external view returns (DNFTDelta[] memory) {
        // 收集该 job 下所有 token 的所有 delta
        uint256[] memory tokens = jobTokens[_jobId];
        uint256 totalCount = 0;
        for (uint i = 0; i < tokens.length; i++) {
            totalCount += jobDeltas[_jobId][tokens[i]].length;
        }
        
        DNFTDelta[] memory result = new DNFTDelta[](totalCount);
        uint idx = 0;
        for (uint i = 0; i < tokens.length; i++) {
            DNFTDelta[] storage deltas = jobDeltas[_jobId][tokens[i]];
            for (uint j = 0; j < deltas.length; j++) {
                result[idx] = deltas[j];
                idx++;
            }
        }
        return result;
    }

    // 辅助：获取 job 的守恒摘要
    function getConservationSummary(uint256 _jobId) 
        external view 
        returns (
            int256 netChirality,          // 应为 0
            uint256 totalPositive,        // +dNFT 价值总和
            uint256 totalNegative,        // -dNFT 价值总和
            bool allAnnihilated,          // 是否全部湮没
            uint256 activeCount           // 未湮没的 dNFT 数量
        )
```

---

## 四、组件 B：conservationHolds() 算法——五层校验

### 4.1 为什么需要多层？

单一 `sum(chirality) == 0` 是不够的。攻击者可以：
- 制造假配对（两个 +dNFT 伪造一个 -dNFT）
- 价值不匹配（+dNFT 价值 $1000 vs -dNFT 价值 $10）
- 跳过中间状态直接湮没（绕过物流验证）
- 配对后只湮没一方（单方面销毁）

因此需要 **5 层递进式校验**：

### 4.2 五层校验逻辑

```solidity
function conservationHolds(DNFTDelta[] memory delta) internal pure 
    returns (bool conserved, string memory reason, ConservationProof memory proof)
{
    // ====== Layer 1：数量守恒（最基础） ======
    int256 netChirality = 0;
    for (uint i = 0; i < delta.length; i++) {
        if (delta[i].newStatus != DNFTStatus.Annihilated)
            netChirality += int256(uint256(delta[i].chirality));
    }
    // 注意：这里统计的是"活跃 dNFT"（未湮没的），不是历史变更
    // 已湮没的 dNFT 不参与计数（它们已从活跃集合中移除）
    if (netChirality != 0) {
        return (false, "L1_FAIL: chirality imbalance", emptyProof());
    }
    proof.netChirality = netChirality;

    // ====== Layer 2：配对完整性 ======
    // 每个 +dNFT 必须有且仅有 1 个同 pairId 的 -dNFT
    // 每个 -dNFT 同理
    if (!checkPairingIntegrity(delta)) {
        return (false, "L2_FAIL: pairing mismatch", proof);
    }
    proof.paired = true;

    // ====== Layer 3：价值守恒 ======
    // Σ(value_i × chirality_i) 必须在容差范围内为 0
    (uint256 posVal, uint256 negVal) = getBilateralValues(delta);
    int256 valueNet = int256(posVal) - int256(negVal);  // 应为 0 或极小值
    uint256 tolerance = posVal > 0 ? posVal / 1_000_000 : 1;  // 0.0001% 容差
    if (abs(valueNet) > int256(tolerance)) {
        return (false, "L3_FAIL: value asymmetry", proof);
    }
    proof.valueAsymmetry = abs(valueNet);

    // ====== Layer 4：状态完备性（顶点序列合法） ======
    // 必须经历完整的：CREATE → TRADE → (IN_TRANSIT) → DELIVERED → ANNIHILATE
    // 不能跳过中间步骤
    if (!checkVertexSequence(delta)) {
        return (false, "L4_FAIL: invalid vertex sequence", proof);
    }
    proof.sequenceValid = true;

    // ====== Layer 5：双世界耦合（Real ↔ Virtual） ======
    // Virtual 层 dNFT 状态变更必须有对应的 Real 层事件签名
    if (!checkRealVirtualCoupling(delta)) {
        return (false, "L5_FAIL: real-virtual decoupling", proof);
    }
    proof.coupled = true;

    // 全部通过
    return (true, "ALL_PASSED", proof);
}
```

### 4.3 各层的具体实现要点

#### Layer 1：数量守恒

```
输入：delta[]（该 Job 所有活跃 dNFT 的当前状态）
算法：
  1. 遍历所有 delta，只取 newStatus != Annihilated 的项（未湮没的）
  2. sum(chirality)
  3. 要求 == 0

边界情况：
  - 空 delta → 通过（Job 无 dNFT 参与，纯服务型任务）
  - 只有 +dNFT 无 -dNFT → 失败
```

**实现注意点**：Layer 1 统计的是"当前活跃"而非"历史上创建过的"。已湮没的 dNFT 不参与计数（它们已经从活跃集合中移除）。

#### Layer 2：配对完整性

```solidity
function checkPairingIntegrity(DNFTDelta[] memory delta) internal pure 
    returns (bool) 
{
    // 用 final state 构建配对映射
    // key = pairId, value = { positiveCount, negativeCount }
    
    for (uint i = 0; i < delta.length; i++) {
        if (delta[i].newStatus == DNFTStatus.Annihilated) continue;
        // ... 统计每个 pairId 下的 +/- 数量
    }
    // 每个 pairId 必须恰好：positive == 1 && negative == 1
}
```

**为什么是 1:1 而不是 N:N？**
因为白皮书的设计是"一笔交易一对 dNFT"。如果一个 Job 涉及多个商品（比如订单包含 3 个 SKU），那就有 3 个独立的 pairId，每对内部仍需 1:1。

#### Layer 3：价值守恒

```
数学表达：
  | Σ(V_+ × (+1)) + Σ(V_- × (-1)) | ≤ ε × max(ΣV_+, ΣV_-)

其中 ε = 0.0001%（百万分之一），用于处理：
  - 浮点到整数的舍入
  - 不同计价单位的微小汇率差
  - 手续费扣减（如果保险费从价值中扣除）
```

**重要决策**：保险费（3%）是否计入价值不平衡？
→ **不计入**。保险费是独立于 dNFT 价值的资金流，属于"费用"而非"资产价值"。
→ 但如果 dNFT.value 本身就包含了保险费调整后的净值，则自然守恒。

#### Layer 4：顶点序列合法性

```
合法序列示例：
  [CREATE] → [TRADE(+dNFT转买家)] → [TRADE(-dNFT转物流)] → 
  [STATUS_UPDATE(InTransit)] → [STATUS_UPDATE(Delivered)] → 
  [ANNIHILATE]

非法序列：
  ❌ [CREATE] → [ANNIHILATE]                    // 跳过交易和物流
  ❌ [CREATE] → [TRADE] → [TRADE] → [TRADE]     // 循环转账
  ❌ 缺少 CREATE 直接出现 TRADE                  // 来源不明
```

这个检查防止了"幽灵 dNFT"——没有经过正规创生流程就进入流通的 dNFT。

#### Layer 5：Real-Virtual 双世界耦合

这是**最难实现也最有创新性的一层**。

```
核心思想：
  每个 Virtual 层的 dNFT 状态变更（链上），必须有对应的 Real 层物理事件签名。

对应关系：
  ┌────────────────────┬───────────────────────────┬────────────────────┐
  │ Virtual 层事件      │ Real 层要求证据            │ 验证方式            │
  ├────────────────────┼───────────────────────────┼────────────────────┤
  │ CREATE (创生)       │ 商家 RWA 商品存在性证明      │ IoT/NFC 初始绑定   │
  │ TRADE (V₁ 顶点)     │ 买卖双方签名              │ ECDSA 签名验证     │
  │ STATUS InTransit   │ 物流扫码/交接             │ 预言机(IoT Oracle) │
  │ STATUS Delivered   │ 送达签收                 │ 预言机 + 买方确认   │
  │ ANNIHILATE (V₂ 顶点)│ 买家 confirmReceipt       │ 买方签名 + 时间戳  │
  └────────────────────┴───────────────────────────┴────────────────────┘

实现方式：
  每个 delta.eventHash 来自于链下预言机的签名：
    eventHash = keccak256(
      abi.encode(realEventId, oracleAddress, signature, offchainDataCID)
    )
  
  FeynmanEvaluator 不验证预言机数据的真实性（那是预言机自己的事），
  但验证"每个 Virtual 变更都有 eventHash ≠ 0"（即都有 Real 侧锚点）
```

**这一层的关键设计选择**：FeynmanEvaluator **信任预言机签名的存在性**，但不验证其内容真实性。这类似于 TLS 证书模型——我们检查证书链完整性，但不重复验证 CA 的身份鉴定流程。

> **⚠ 适用范围：纯数字商品豁免 L5**
> L5 仅对**含实物 RWA 的交易**强制执行。纯数字商品（NFT、电子书、API 凭证、软件许可证、链上凭证等）交付本身在链上可验证（哈希匹配 / 合约执行结果 / 买方签名），evaluate() 只需跑 L1–L4 即可完成放行，详见第十一节"开放问题"中的 **已确定设计决策（Q1）**。

---

## 五、组件 C：ERC-8183 Job ↔ Escrow Order 映射

### 5.1 两套系统的 ID 问题

| 系统 | 主键 | 状态机 | 触发方 |
|------|------|--------|-------|
| **ERC-8183 Job** | jobId (uint256) | Open → Funded → Submitted → Terminal | Client/Provider/Evaluator |
| **Escrow Order** | orderId (bytes32) | Created → FUNDED → SHIPPED → DELIVERED → COMPLETED | Buyer/Merchant/Logistics |

这两者是**同一笔商业交易的两个视角**：

```
ERC-8183 视角（Agent 经济通用协议）：
  Client(买家Agent) 创建 Job → fund(付款) → Provider(商家Agent) submit(deliverable) 
  → Evaluator(FeynmanEvaluator) evaluate → complete/reject

Escrow 视角（费曼图专用执行层）：
  Buyer placeOrder(付款+dNFT) → Merchant acceptOrder → Logistics confirmDelivery 
  → Buyer confirmReceipt(湮没) → Fund release
```

### 5.2 映射方案：JobBridge 合约

```solidity
contract JobBridge {
    // 双向映射
    mapping(uint256 => bytes32) public jobToOrder;    // ERC-8183 jobId → Escrow orderId
    mapping(bytes32 => uint256) public orderToJob;     // Escrow orderId → ERC-8183 jobId
    
    // 由 Escrow.placeOrder 时自动注册
    function registerMapping(uint256 _jobId, bytes32 _orderId) 
        external onlyEscrow 
    {
        jobToOrder[_jobId] = _orderId;
        orderToJob[_orderId] = _jobId;
        emit MappingCreated(_jobId, _orderId);
    }
    
    // FeynmanEvaluator 调用：从 jobId 拿到 orderId，再去 DNFTRegistry 取 delta
    function resolveOrderId(uint256 _jobId) external view returns (bytes32) {
        require(jobToOrder[_jobId] != bytes32(0), "No mapping");
        return jobToOrder[_jobId];
    }
}
```

### 5.3 谁来调用 evaluate()？

ERC-8183 定义了两种触发模式：

**模式 A（推荐）：Hook 自动触发**

```
ERC-8183 Hook 机制允许在 submit() 后自动挂载自定义逻辑：

submit() 执行后:
  1. Job 状态 → Submitted
  2. Hook 自动触发 FeynmanEvaluator.evaluate(jobId, deliverable)
  3. evaluate 内部:
     a. 从 JobBridge 拿到 orderId
     b. 从 DNFTRegistry.getDelta(orderId) 取全量 dNFT 变更
     c. 跑 conservationHolds()
     d. 通过 → 调用 complete(jobId, proof)
     e. 不通过 → 调用 reject(jobId, proof + reason)
```

**模式 B：手动触发（fallback）**

如果 Hook 未配置或 Hook 执行失败，任何人可以调用 `evaluate()` 作为独立操作。但需加 access control：只有 Evaluator role 可以调用 complete/reject。

---

## 六、组件 D：deliverable 格式定义

### 6.1 Provider 提交什么？

在费曼图模型的语境下，ERC-8183 的 `submit(deliverable)` 不是提交"文件"或"代码"，而是提交**交易完成的数学声明**：

```solidity
struct FeynmanDeliverable {
    uint256 orderId;              // 对应的 Escrow 订单
    uint256 dnftPairId;          // 涉及的 dNFT 配对 ID
    DNFTStatus expectedFinalState; // 声称的最终状态（应为 Annihilated）
    bytes32 realWorldEvidenceCID; // Real 层证据的 IPFS/Arweave CID
                                // （如：收货照片哈希、IoT 送达日志等）
    bytes32[] oracleSignatures;   // 预言机签名数组（物流各节点的签名）
    bytes32 merkleRoot;          // 所有链下证据的 Merkle root
                                // （用于 ZKP 验证时不必暴露全部数据）
}
```

### 6.2 deliverable 与 dNFT delta 的交叉验证

```
evaluate(jobId, deliverable) 流程：

1. 解码 deliverable → 得到 FeynmanDeliverable 结构体
2. 从 DNFTRegistry.getDelta(orderId) 取链上记录的真实 delta
3. 交叉比对：
   a. deliverable.dnftPairId ∈ delta 中的 pairIds?
   b. deliverable.expectedFinalState == Annihilated?
   c. deliverable.oracleSignatures 能覆盖 delta 中所有 Real-Virtual 耦合点?
4. 如果交叉验证通过 → 进入 conservationHolds()
5. 如果交叉验证不通过 → 直接 reject("deliverable mismatch")
```

---

## 七、完整 evaluate() 流程图

```
┌─────────────────────────────────────────────────────────────────┐
│               FeynmanEvaluator.evaluate()                       │
│                                                                 │
│  输入: (jobId, deliverable_bytes)                               │
│                                                                 │
│  Step 1: 解码 & 映射                                            │
│  ├─ ABI decode deliverable → FeynmanDeliverable                │
│  ├─ JobBridge.resolveOrderId(jobId) → orderId                  │
│  └─ 验证 jobId 状态 == Submitted (ERC-8183)                     │
│                                                                 │
│  Step 2: 数据获取                                               │
│  ├─ DNFTRegistry.getDelta(orderId) → delta[]                   │
│  ├─ DynamicNFT.batchGetInfo(tokenIds) → 当前实时状态            │
│  └─ (可选) 从链下预取预言机签名                                 │
│                                                                 │
│  Step 3: 交叉验证 (deliverable vs on-chain state)              │
│  ├─ pairId 匹配?                                                │
│  ├─ 声明状态 == 链上实际?                                        │
│  └─ oracle 签名覆盖度足够?                                      │
│         ↓ No                                                   │
│      reject("deliverable mismatch")                             │
│                                                                 │
│  Step 4: conservationHolds(delta[]) — 五层校验                  │
│  ├─ L1: 数量守恒 Σ(chirality) = 0                              │
│  ├─ L2: 配对完整性 (1:1 pairing)                               │
│  ├─ L3: 价值守恒 |Σ(value×chirality)| ≤ ε                       │
│  ├─ L4: 顶点序列合法 (CREATE→TRADE→...→ANNIHILATE)            │
│  └─ L5: Real-Virtual 耦合 (每个 Virtual 有 Real anchor)         │
│         ↓ Fail at any layer                                    │
│      reject(proof + layer + reason)                             │
│                                                                 │
│  Step 5: 生成证明 & 执行                                        │
│  ├─ proof = keccak256(delta + result + blockhash)              │
│  ├─ DNFTRegistry.markClosed(orderId, proof)                    │
│  ├─ ERC8183Job.complete(jobId, proof)  ← 放款给 Provider       │
│  └─ emit Evaluated(jobId, true, proof)                          │
│                                                                 │
│  输出: (passed=true, proof)                                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## 八、Gas 优化策略（工程关键）

### 8.1 Gas 瓶颈在哪？

| 操作 | 预估 Gas | 说明 |
|------|---------|------|
| DNFTRegistry.getDelta() 读大量 storage | ~50k–200k | 取决于 delta 条数 |
| Layer 2 配对检查（嵌套循环） | ~30k–80k | O(n²) 配对查找 |
| Layer 4 序列检查 | ~20k | 需排序后遍历 |
| Layer 5 签名验证 | ~100k+/sig | 每个预言机签名一次 ecrecover |
| **总计** | **200k–600k+** | **可能触及 block gas limit** |

### 8.2 优化方案

**方案 1：增量验证（推荐）**

不在最后一步一次性跑完 5 层，而是在**每个顶点事件发生时就做增量校验**：

```
placeOrder() 时（V₁ 顶点）：
  → Layer 1 检查：新加入的 dNFT chirality 和仍为 0 ✓
  → Layer 4 检查：CREATE 之前已有记录 ✓

confirmDelivery() 时：
  → Layer 5 检查：IoT 预言机签名存在 ✓
  → Layer 4 检查：状态转换 IN_TRANSIT → Delivered 合法 ✓

confirmReceipt() 时（V₂ 顶点）：
  → Layer 1 检查：湮没后 chirality 和回归 0 ✓
  → Layer 2 检查：配对方同时标记 Annihilated ✓
  → Layer 3 检查：价值净值为 0 ✓
  → Layer 4 检查：完整序列 ✓
```

evaluate() 最后只需做一个**轻量的最终一致性检查**（~50k gas），而不是从头重跑全部 5 层。

**方案 2：链下计算 + ZKP 证明**

将 5 层校验的全部计算移到链下，生成 SNARK/STARK 证明。链上只需 verify Proof（~200k gas 固定成本，与 delta 数量无关）。这与 ERC-8126 PDV（私有数据验证）的理念一致。

**方案 3：分阶段提交**

```
Phase 1（MVP）：只实现 L1 + L2（数量 + 配对），gas ~80k
Phase 2：加 L3（价值守恒），额外 ~30k
Phase 3：加 L4（序列检查），额外 ~40k
Phase 4：加 L5（双世界耦合），依赖预言机成熟度
```

---

## 九、安全分析与攻击面

### 9.1 攻击向量矩阵

| 攻击向量 | 影响层级 | 防御措施 | 剩余风险 |
|---------|---------|---------|---------|
| **虚假 dNFT 创生**（伪造 +dNFT/-dNFT 对） | L1/L2 | DNFTRegistry 只接受 DynamicNFT 合约授权写入；createPair 需要 RWA 绑定证明 | 低（需攻破 DynamicNFT） |
| **价值操纵**（+dNFT 价值 $1000, -dNFT 价值 $10） | L3 | 价值由创生时的预言机报价锁定，后续不可变；Layer 3 容差检查 | 低 |
| **跳步攻击**（直接 CREATE→ANNIHILATE） | L4 | Layer 4 顶点序列强制检查；缺少中间状态的拒绝 | 低 |
| **预言机作恶**（伪造物流签名） | L5 | 多预言机冗余（≥3 个独立源）；Staking 惩罚机制 | 中（依赖预言机网络质量） |
| **Evaluator 合约本身被替换** | 全局 | ERC-8183 的 Evaluator 地址在 Job 创建时锁定不可变；Proxy 升级需 Timelock | 低 |
| **Front-running evaluate()** | 时序 | evaluate() 结果是确定性的（相同 input → 相同 output）；不存在 MEV 机会 | 无 |
| **DoS（恶意制造海量 delta）** | Gas | delta 数量上限（如单 Job ≤ 50 个 dNFT）；超出拒绝 | 低 |

### 9.2 与 ERC-8183 自身风险的叠加

ERC-8183 安全条款明确警告："**malicious evaluator can arbitrarily complete or reject**"。

FeynmanEvaluator **消除了主观作恶的可能**（裁决由数学决定），但引入了新的风险：

| 新风险 | 概率 | 影响 | 缓解 |
|--------|------|------|------|
| 守恒算法 bug 导致合法交易被 reject | 低 | 高（资金锁死）| 形式化验证 + 多审计 + TimeLock 紧急覆盖 |
| 守恒算法漏洞导致非法交易被 complete | 极低 | 极高（资金错误释放）| 11.7 保险池 staking 覆盖损失 |
| Gas 飙升导致 evaluate() 总是 OOM | 中 | 中（Job 卡在 Submitted）| 方案 1 增量验证 + fallback 到手动仲裁 |

**最重要的缓解措施就是白皮书 11.7 的保险池/staking 机制**：即使 Evaluator 数学逻辑被突破，经济层面仍有担保金覆盖损失。

---

## 十、实现路线图（Phasing）

```
Phase 0（基础设施，前置依赖）
  ├── DynamicNFT.sol 完善（pairId/chirality/status 机）
  ├── DNFTRegistry.sol 开发 & 部署
  ├── JobBridge.sol 开发 & 部署
  └── ERC-8183 Draft 实现（等 Final 再锁定接口）

Phase 1（MVP：最小可运行 Evaluator）
  ├── FeynmanEvaluator v0.1：仅 L1+L2（数量+配对）
  ├── deliverable 格式 v0.1（简化版，不含 ZKP）
  ├── 手动触发模式（非 Hook 自动）
  └── 目标：能正确判断"正常交易 pass / 明显异常 reject"

Phase 2（生产就绪）
  ├── 加 L3（价值守恒）
  ├── 加 L4（顶点序列）
  ├── deliverable 格式 v1.0（含 oracle signatures）
  ├── Hook 自动触发集成
  └── Gas 基准测试 & 优化

Phase 3（完整版）
  ├── 加 L5（双世界耦合 + 预言机签名验证）
  ├── ZKP 证明模式（可选替代全链上计算）
  ├── 11.7 保险池 staking 集成
  └── 形式化验证（Certora/K）

Phase 4（生态扩展）
  ├── 多链部署（Base/Arbitrum/Optimism L2）
  ├── 与 ERC-8004 声誉回写对接
  └── Kleros 仲裁作为 L5 fallback
```

---

## 十一、开放问题（待研究/待决策）

> **✅ 已确定的设计决策（从开放问题移出）**：
> 纯数字商品**豁免 L5**（双世界耦合层）。数字商品（NFT、电子书、API 凭证、软件许可证、链上凭证等）的交付本身在链上可验证——通过合约执行结果、文件哈希匹配或买方数字签名确认即可，不需物理世界 RWA 事件锚点。
> L5 仅对**含实物 RWA 的交易**强制执行。这让纯数字商品交易的 evaluate() 只需跑 L1–L4，gas 成本更低、部署更快，与 ERC-8183 允许"合约 Evaluator 处理确定性任务"的设计完全对齐。
>
> | 校验层 | 实物 RWA 交易 | 纯数字商品交易 |
> | :---: | :---: | :---: |
> | L1 数量守恒 | ✅ | ✅ |
> | L2 配对完整 | ✅ | ✅ |
> | L3 价值守恒 | ✅ | ✅ |
> | L4 顶点序列 | ✅ | ✅ |
> | **L5 双世界耦合** | **✅ 必须有预言机锚点** | **⚡ 豁免** |

| # | 问题 | 影响 | 建议 |
|---|------|------|------|
| 1 | **部分退货场景下的守恒怎么算？** | 退 3 件中的 1 件，dNFT 如何分割？ | 可能需要引入"dNFT 分数化"机制或"子配对"概念 |
| 2 | **多商家联合订单（一个 Job 含多个商家的 dNFT）** | 守恒范围是 per-job 还是 per-merchant？ | Per-job（一个 Feynman 图可以有多个入射/出射粒子） |
| 3 | **跨链场景下 δ 怎么收集？** | 商品在一个链，支付在另一个链 | 使用跨链消息传递（CCIP/LayerZero）同步 delta；或限制单链内闭环 |
| 4 | **conservationHolds 运行在 EVM 上还是链下？** | 决定安全性模型和 Gas 成本 | Phase 1-3 链上；Phase 4 可选 ZKP 链下 |

---

## 十二、与白皮书其他章节的衔接关系

```
第 3 章（形式化框架）──→ 提供 Σ(dNFT)=0 的数学定义
    │
    ▼
附录 A（数学形式化）  ─→ 提供顶点规则、传播子函数的形式化描述
    │
    ▼
第 7 章（智能合约架构）──→ DynamicNFT.sol / Escrow.sol 是本合约的数据提供方
    │
    ▼
第 11.7（保险池）     ─→ 为 Evaluator 提供 staking 经济背书
    │
    ▼
第 11.8（本文档主题）──→ FeynmanEvaluator：把数学定义变成可执行代码
    │
    ▼
ERC-8183 标准         ─→ 定义 evaluate/complete/reject 的接口契约
```

---

*文档版本：v1.0 | 日期：2026-07-12 | 状态：技术分析稿（非最终实现）*
