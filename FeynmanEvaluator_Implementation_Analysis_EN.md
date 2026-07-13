# Feynman Evaluator Contract Implementation Analysis (The Mathematical Kernel for ERC-8183 Evaluator Interface)

> This document provides an in-depth deconstruction of the FeynmanEvaluator contract from Section 11.8 of the whitepaper: from current pseudocode to a complete implementable design.
> Core question: How to translate the dNFT conservation law Σ(dNFT) = 0 — a mathematical constraint — into verifiable execution logic on-chain in Solidity.

---

## I. Problem Framing: Section 11.8 Current State vs. Implementation Gaps

### 1.1 What the Whitepaper Currently Contains (Pseudocode Only)

```
interface IFeynmanEvaluator {
    function evaluate(uint256 jobId, bytes calldata deliverable)
        external returns (bool passed, bytes32 proof);
}
// Inside evaluate():
//   1. delta = DNFTRegistry.getDelta(jobId)     ← What is this?
//   2. conserved = conservationHolds(delta)      ← How is it computed?
//   3. proof = keccak256(abi.encode(delta, ts))
//   4. Pass → complete / Fail → reject
```

### 1.2 Four Critical Gaps

| # | Gap | Whitepaper Location | Description |
|---|--------|---------------|------|
| **A** | **DNFTRegistry contract does not exist** | Line 3826 only lists filename `contracts/DNFTRegistry.sol`, no code | Data source for getDelta(jobId) |
| **B** | **conservationHolds() algorithm undefined** | Line 3283, a single-line invocation | How to check Σ(dNFT)=0 inside a contract? |
| **C** | **Mapping to ERC-8183 Job is unclear** | Is jobId from ERC-8183 or from Escrow? | How do the two ID systems interoperate? |
| **D** | **deliverable parameter format undefined** | bytes calldata deliverable | What does the Provider submit? What does the Evaluator verify against? |

These four gaps are the implementation details this document will fill in, one by one.

---

## II. Core Concept Translation: From Physics to Solidity

### 2.1 What Exactly Does "Σ(dNFT)=0" Verify?

Returning to the whitepaper's physical definition (lines 1437–1444):

```
Before creation:     Σ(dNFT) = 0
After creation:      Σ(dNFT) = (+1) + (-1) = 0       // Merchant mints dNFT pair
After trade contract: Σ(dNFT) = (+1_buyer) + (-1_logistics) = 0  // V₁ vertex
After annihilation:  Σ(dNFT) = 0                        // V₂ vertex, pair disappears
```

**Key insight**: Σ(dNFT)=0 is not a global invariant (multiple transactions can coexist in the system); it is the **closure condition for a single transaction** — the algebraic sum of dNFTs involved in a transaction must return to zero by the end of the transaction.

Translated into contract language:

> **conservationHolds(jobId)** = Check whether all dNFT state changes involved in this Job form a "closed Feynman diagram" — i.e., every +dNFT has found its corresponding -dNFT pair and completed annihilation.

### 2.2 dNFT State Machine (Already Defined)

Already defined in whitepaper lines 1670–1700 and DynamicNFT.sol (appendix code starting at line 3886):

```
dNFT state flow:
  Created → Locked → InTransit → Delivered → Annihilated
                    ↘ Refunded

Each dNFT has:
  tokenId: uint256          // Unique identifier
  pairId: uint256           // Pair ID (+dNFT and -dNFT share the same pairId)
  chirality: int8           // Chirality: +1 or -1
  owner: address            // Current holder
  status: enum Status       // Status
  value: uint256            // Anchored value (commodity price)
  pairedTokenId: uint256    // Paired token ID
```

**Equivalent formulations of the conservation law**:

| Conservation Dimension | Physical Meaning | Solidity Check |
|---------|---------|------------------|
| Quantity conservation | Σ(chirality) = 0 | `sum(d.chirality for d in job_dNFTs) == 0` |
| Pairing conservation | Every +dNFT has a unique -dNFT pair | Every +dNFT.pairId finds a match among -dNFTs |
| Value conservation | Σ(value × chirality) ≈ 0 | Net positive/negative value sum approaches zero (rounding tolerance ≤ 1e-12) |
| State conservation | At transaction end, all dNFTs are annihilated | `all(d.status == Annihilated)` |
| Dual-world conservation | ΔReal + ΔVirtual = 0 | Real-layer asset transfer amount = Virtual-layer credit change |

---

## III. Component A: DNFTRegistry Design (Data Source)

### 3.1 Responsibilities

DNFTRegistry is a **read-only data source** for FeynmanEvaluator. It does not hold dNFTs (that is the DynamicNFT contract's job); instead, it records **which dNFTs each Job/Order involves, and their state change trajectories**.

### 3.2 Core Data Structures

```solidity
contract DNFTRegistry {
    DynamicNFT public dnftContract;

    // Single dNFT state change record
    struct DNFTDelta {
        uint256 tokenId;
        int8 chirality;          // +1 or -1
        address fromOwner;       // Holder before change
        address toOwner;         // Holder after change
        DNFTStatus oldStatus;
        DNFTStatus newStatus;
        uint256 value;           // Anchored value
        uint256 timestamp;
        bytes32 eventHash;       // On-chain event hash that triggered this change (vertex signature)
        VertexType vertexType;   // CREATE / TRADE / ANNIHILATE / RETURN
    }

    // Complete dNFT change snapshot for a Job
    struct JobSnapshot {
        uint256 jobId;
        uint256[] tokenIds;              // All dNFTs involved in this job
        mapping(uint256 => DNFTDelta[]) deltasByToken;  // Change history per token
        uint256 totalPositiveValue;      // Total +dNFT value
        uint256 totalNegativeValue;      // Total -dNFT value
        bool isClosed;                   // Whether the job is closed
        bytes32 closureProof;            // Closure proof hash
    }

    enum VertexType { CREATE, TRADE, ANNIHILATE, RETURN }
}
```

### 3.3 Key Functions

```solidity
    // Called by the Escrow contract at key milestones to record dNFT changes
    function recordDelta(
        uint256 _jobId,
        uint256 _tokenId,
        VertexType _vertexType,
        bytes32 _eventHash
    ) external onlyAuthorized {
        // Read current state from DynamicNFT contract
        (int8 chirality, , address owner, DNFTStatus status, uint256 value, , ) 
            = dnftContract.getDNFTInfo(_tokenId);
        
        // Derive from/to (from previous delta)
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

    // Core interface called by FeynmanEvaluator
    function getDelta(uint256 _jobId) external view returns (DNFTDelta[] memory) {
        // Collect all deltas for all tokens under this job
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

    // Helper: get conservation summary for a job
    function getConservationSummary(uint256 _jobId) 
        external view 
        returns (
            int256 netChirality,          // Should be 0
            uint256 totalPositive,        // Sum of +dNFT values
            uint256 totalNegative,        // Sum of -dNFT values
            bool allAnnihilated,          // Whether all are annihilated
            uint256 activeCount           // Number of non-annihilated dNFTs
        )
```

---

## IV. Component B: conservationHolds() Algorithm — Five-Layer Verification

### 4.1 Why Multiple Layers?

A single `sum(chirality) == 0` is insufficient. An attacker could:
- Fabricate fake pairs (two +dNFTs faking a single -dNFT)
- Mismatch values (+dNFT valued at $1000 vs. -dNFT valued at $10)
- Skip intermediate states and annihilate directly (bypassing logistics verification)
- Annihilate only one side after pairing (unilateral destruction)

Hence the need for **5 progressive verification layers**:

### 4.2 Five-Layer Verification Logic

```solidity
function conservationHolds(DNFTDelta[] memory delta) internal pure 
    returns (bool conserved, string memory reason, ConservationProof memory proof)
{
    // ====== Layer 1: Quantity Conservation (fundamental) ======
    int256 netChirality = 0;
    for (uint i = 0; i < delta.length; i++) {
        if (delta[i].newStatus != DNFTStatus.Annihilated)
            netChirality += int256(uint256(delta[i].chirality));
    }
    // Note: This counts "active dNFTs" (non-annihilated), not historical changes
    // Annihilated dNFTs are excluded (they have been removed from the active set)
    if (netChirality != 0) {
        return (false, "L1_FAIL: chirality imbalance", emptyProof());
    }
    proof.netChirality = netChirality;

    // ====== Layer 2: Pairing Integrity ======
    // Every +dNFT must have exactly 1 -dNFT with the same pairId
    // Likewise every -dNFT
    if (!checkPairingIntegrity(delta)) {
        return (false, "L2_FAIL: pairing mismatch", proof);
    }
    proof.paired = true;

    // ====== Layer 3: Value Conservation ======
    // Σ(value_i × chirality_i) must be 0 within tolerance
    (uint256 posVal, uint256 negVal) = getBilateralValues(delta);
    int256 valueNet = int256(posVal) - int256(negVal);  // Should be 0 or negligible
    uint256 tolerance = posVal > 0 ? posVal / 1_000_000 : 1;  // 0.0001% tolerance
    if (abs(valueNet) > int256(tolerance)) {
        return (false, "L3_FAIL: value asymmetry", proof);
    }
    proof.valueAsymmetry = abs(valueNet);

    // ====== Layer 4: State Completeness (Vertex Sequence Legality) ======
    // Must go through the full sequence: CREATE → TRADE → (IN_TRANSIT) → DELIVERED → ANNIHILATE
    // Intermediate steps cannot be skipped
    if (!checkVertexSequence(delta)) {
        return (false, "L4_FAIL: invalid vertex sequence", proof);
    }
    proof.sequenceValid = true;

    // ====== Layer 5: Dual-World Coupling (Real ↔ Virtual) ======
    // Every Virtual-layer dNFT state change must have a corresponding Real-layer event signature
    if (!checkRealVirtualCoupling(delta)) {
        return (false, "L5_FAIL: real-virtual decoupling", proof);
    }
    proof.coupled = true;

    // All passed
    return (true, "ALL_PASSED", proof);
}
```

### 4.3 Implementation Details for Each Layer

#### Layer 1: Quantity Conservation

```
Input: delta[] (current state of all active dNFTs for this Job)
Algorithm:
  1. Iterate all deltas; only include items where newStatus != Annihilated (non-annihilated)
  2. sum(chirality)
  3. Require == 0

Edge cases:
  - Empty delta → pass (Job involves no dNFTs, pure service task)
  - Only +dNFTs, no -dNFTs → fail
```

**Implementation note**: Layer 1 counts "currently active" dNFTs, not "historically created" ones. Annihilated dNFTs are excluded from the count (they have already been removed from the active set).

#### Layer 2: Pairing Integrity

```solidity
function checkPairingIntegrity(DNFTDelta[] memory delta) internal pure 
    returns (bool) 
{
    // Build pairing map from final state
    // key = pairId, value = { positiveCount, negativeCount }
    
    for (uint i = 0; i < delta.length; i++) {
        if (delta[i].newStatus == DNFTStatus.Annihilated) continue;
        // ... count +/- under each pairId
    }
    // Each pairId must have exactly: positive == 1 && negative == 1
}
```

**Why 1:1 and not N:N?**
Because the whitepaper's design is "one transaction, one dNFT pair." If a Job involves multiple goods (e.g., an order containing 3 SKUs), there are 3 independent pairIds, and each pair must still be internally 1:1.

#### Layer 3: Value Conservation

```
Mathematical expression:
  | Σ(V_+ × (+1)) + Σ(V_- × (-1)) | ≤ ε × max(ΣV_+, ΣV_-)

Where ε = 0.0001% (one in a million), used to handle:
  - Floating-point to integer rounding
  - Minor exchange rate differences between pricing units
  - Fee deductions (if insurance premiums are deducted from value)
```

**Important decision**: Does the insurance premium (3%) count as value imbalance?
→ **No.** The premium is a capital flow independent of dNFT value, classified as a "fee" rather than "asset value."
→ However, if dNFT.value already includes the net value after premium adjustment, conservation holds naturally.

#### Layer 4: Vertex Sequence Legality

```
Valid sequence example:
  [CREATE] → [TRADE(+dNFT→buyer)] → [TRADE(-dNFT→logistics)] → 
  [STATUS_UPDATE(InTransit)] → [STATUS_UPDATE(Delivered)] → 
  [ANNIHILATE]

Invalid sequences:
  ❌ [CREATE] → [ANNIHILATE]                    // Skipped trade and logistics
  ❌ [CREATE] → [TRADE] → [TRADE] → [TRADE]     // Circular transfers
  ❌ TRADE without preceding CREATE              // Unknown origin
```

This check prevents "ghost dNFTs" — dNFTs that enter circulation without going through the formal creation process.

#### Layer 5: Real-Virtual Dual-World Coupling

This is the **hardest to implement and most innovative layer**.

```
Core idea:
  Every Virtual-layer dNFT state change (on-chain) must have a corresponding Real-layer physical event signature.

Correspondence:
  ┌────────────────────┬───────────────────────────┬────────────────────┐
  │ Virtual-layer Event │ Real-layer Required Evidence  │ Verification Method │
  ├────────────────────┼───────────────────────────┼────────────────────┤
  │ CREATE             │ Merchant RWA commodity existence proof │ IoT/NFC initial binding │
  │ TRADE (V₁ vertex)  │ Buyer and seller signatures           │ ECDSA signature verification │
  │ STATUS InTransit   │ Logistics scan/handover               │ Oracle (IoT Oracle) │
  │ STATUS Delivered   │ Delivery confirmation                 │ Oracle + buyer confirmation │
  │ ANNIHILATE (V₂)    │ Buyer confirmReceipt                  │ Buyer signature + timestamp │
  └────────────────────┴───────────────────────────┴────────────────────┘

Implementation approach:
  Each delta.eventHash comes from an off-chain oracle signature:
    eventHash = keccak256(
      abi.encode(realEventId, oracleAddress, signature, offchainDataCID)
    )
  
  FeynmanEvaluator does not verify the truthfulness of oracle data (that is the oracle's responsibility),
  but verifies that "every Virtual change has eventHash ≠ 0" (i.e., has a Real-side anchor point)
```

**Key design choice for this layer**: FeynmanEvaluator **trusts the existence of oracle signatures** but does not verify their content truthfulness. This is analogous to the TLS certificate model — we check certificate chain integrity without re-verifying the CA's identity verification process.

> **⚠ Scope: Pure Digital Goods Exempt from L5**
> L5 is enforced **only for transactions involving physical RWA**. Pure digital goods (NFTs, e-books, API credentials, software licenses, on-chain credentials, etc.) have delivery that is verifiable on-chain (hash matching / contract execution results / buyer signatures), so evaluate() only needs to run L1–L4 for clearance. See Section XI "Open Questions" under **Confirmed Design Decision (Q1)**.

---

## V. Component C: ERC-8183 Job ↔ Escrow Order Mapping

### 5.1 The Two-System ID Problem

| System | Primary Key | State Machine | Triggering Party |
|------|------|--------|-------|
| **ERC-8183 Job** | jobId (uint256) | Open → Funded → Submitted → Terminal | Client/Provider/Evaluator |
| **Escrow Order** | orderId (bytes32) | Created → FUNDED → SHIPPED → DELIVERED → COMPLETED | Buyer/Merchant/Logistics |

These two are **two perspectives on the same commercial transaction**:

```
ERC-8183 Perspective (General Agent Economy Protocol):
  Client(buyer Agent) creates Job → fund(payment) → Provider(merchant Agent) submit(deliverable) 
  → Evaluator(FeynmanEvaluator) evaluate → complete/reject

Escrow Perspective (Feynman-diagram-specific execution layer):
  Buyer placeOrder(pay+dNFT) → Merchant acceptOrder → Logistics confirmDelivery 
  → Buyer confirmReceipt(annihilate) → Fund release
```

### 5.2 Mapping Solution: JobBridge Contract

```solidity
contract JobBridge {
    // Bidirectional mapping
    mapping(uint256 => bytes32) public jobToOrder;    // ERC-8183 jobId → Escrow orderId
    mapping(bytes32 => uint256) public orderToJob;     // Escrow orderId → ERC-8183 jobId
    
    // Auto-registered when Escrow.placeOrder is called
    function registerMapping(uint256 _jobId, bytes32 _orderId) 
        external onlyEscrow 
    {
        jobToOrder[_jobId] = _orderId;
        orderToJob[_orderId] = _jobId;
        emit MappingCreated(_jobId, _orderId);
    }
    
    // Called by FeynmanEvaluator: get orderId from jobId, then fetch delta from DNFTRegistry
    function resolveOrderId(uint256 _jobId) external view returns (bytes32) {
        require(jobToOrder[_jobId] != bytes32(0), "No mapping");
        return jobToOrder[_jobId];
    }
}
```

### 5.3 Who Calls evaluate()?

ERC-8183 defines two trigger modes:

**Mode A (Recommended): Hook Auto-Trigger**

```
ERC-8183 Hook mechanism allows auto-mounting custom logic after submit():

After submit() executes:
  1. Job state → Submitted
  2. Hook auto-triggers FeynmanEvaluator.evaluate(jobId, deliverable)
  3. Inside evaluate:
     a. Get orderId from JobBridge
     b. Fetch full dNFT changes from DNFTRegistry.getDelta(orderId)
     c. Run conservationHolds()
     d. Pass → call complete(jobId, proof)
     e. Fail → call reject(jobId, proof + reason)
```

**Mode B: Manual Trigger (fallback)**

If Hook is not configured or Hook execution fails, anyone can call `evaluate()` as a standalone operation. However, access control is required: only the Evaluator role can call complete/reject.

---

## VI. Component D: deliverable Format Definition

### 6.1 What Does the Provider Submit?

In the context of the Feynman diagram model, ERC-8183's `submit(deliverable)` is not about submitting "files" or "code," but rather submitting a **mathematical declaration of transaction completion**:

```solidity
struct FeynmanDeliverable {
    uint256 orderId;              // Corresponding Escrow order
    uint256 dnftPairId;          // Involved dNFT pair ID
    DNFTStatus expectedFinalState; // Claimed final state (should be Annihilated)
    bytes32 realWorldEvidenceCID; // IPFS/Arweave CID of Real-layer evidence
                                // (e.g., receipt photo hash, IoT delivery logs, etc.)
    bytes32[] oracleSignatures;   // Oracle signature array (signatures from logistics nodes)
    bytes32 merkleRoot;          // Merkle root of all off-chain evidence
                                // (for ZKP verification without exposing all data)
}
```

### 6.2 Cross-Verification of deliverable Against dNFT Delta

```
evaluate(jobId, deliverable) flow:

1. Decode deliverable → get FeynmanDeliverable struct
2. Fetch on-chain recorded real deltas from DNFTRegistry.getDelta(orderId)
3. Cross-check:
   a. deliverable.dnftPairId ∈ pairIds in delta?
   b. deliverable.expectedFinalState == Annihilated?
   c. deliverable.oracleSignatures cover all Real-Virtual coupling points in delta?
4. If cross-verification passes → enter conservationHolds()
5. If cross-verification fails → directly reject("deliverable mismatch")
```

---

## VII. Complete evaluate() Flowchart

```
┌─────────────────────────────────────────────────────────────────┐
│               FeynmanEvaluator.evaluate()                       │
│                                                                 │
│  Input: (jobId, deliverable_bytes)                              │
│                                                                 │
│  Step 1: Decode & Map                                           │
│  ├─ ABI decode deliverable → FeynmanDeliverable                │
│  ├─ JobBridge.resolveOrderId(jobId) → orderId                  │
│  └─ Verify jobId state == Submitted (ERC-8183)                  │
│                                                                 │
│  Step 2: Data Fetching                                          │
│  ├─ DNFTRegistry.getDelta(orderId) → delta[]                   │
│  ├─ DynamicNFT.batchGetInfo(tokenIds) → current live state     │
│  └─ (Optional) Pre-fetch oracle signatures off-chain            │
│                                                                 │
│  Step 3: Cross-Verification (deliverable vs on-chain state)    │
│  ├─ pairId match?                                               │
│  ├─ Claimed state == actual on-chain state?                     │
│  └─ Oracle signature coverage sufficient?                       │
│         ↓ No                                                   │
│      reject("deliverable mismatch")                             │
│                                                                 │
│  Step 4: conservationHolds(delta[]) — Five-Layer Verification   │
│  ├─ L1: Quantity conservation Σ(chirality) = 0                 │
│  ├─ L2: Pairing integrity (1:1 pairing)                        │
│  ├─ L3: Value conservation |Σ(value×chirality)| ≤ ε             │
│  ├─ L4: Vertex sequence valid (CREATE→TRADE→...→ANNIHILATE)   │
│  └─ L5: Real-Virtual coupling (every Virtual has a Real anchor) │
│         ↓ Fail at any layer                                    │
│      reject(proof + layer + reason)                             │
│                                                                 │
│  Step 5: Generate Proof & Execute                               │
│  ├─ proof = keccak256(delta + result + blockhash)              │
│  ├─ DNFTRegistry.markClosed(orderId, proof)                    │
│  ├─ ERC8183Job.complete(jobId, proof)  ← Release funds to Provider │
│  └─ emit Evaluated(jobId, true, proof)                          │
│                                                                 │
│  Output: (passed=true, proof)                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## VIII. Gas Optimization Strategies (Engineering Critical)

### 8.1 Where Are the Gas Bottlenecks?

| Operation | Estimated Gas | Description |
|------|---------|------|
| DNFTRegistry.getDelta() reads large storage | ~50k–200k | Depends on number of deltas |
| Layer 2 pairing check (nested loop) | ~30k–80k | O(n²) pair search |
| Layer 4 sequence check | ~20k | Requires sort then iterate |
| Layer 5 signature verification | ~100k+/sig | One ecrecover per oracle signature |
| **Total** | **200k–600k+** | **May approach block gas limit** |

### 8.2 Optimization Approaches

**Approach 1: Incremental Verification (Recommended)**

Instead of running all 5 layers in one go at the final step, **do incremental checks at each vertex event occurrence**:

```
At placeOrder() (V₁ vertex):
  → Layer 1 check: new dNFT chirality sum still 0 ✓
  → Layer 4 check: CREATE has prior record ✓

At confirmDelivery():
  → Layer 5 check: IoT oracle signature exists ✓
  → Layer 4 check: state transition IN_TRANSIT → Delivered is valid ✓

At confirmReceipt() (V₂ vertex):
  → Layer 1 check: chirality sum returns to 0 after annihilation ✓
  → Layer 2 check: paired side also marked Annihilated ✓
  → Layer 3 check: net value is 0 ✓
  → Layer 4 check: complete sequence ✓
```

evaluate() only needs a final **lightweight consistency check** (~50k gas) rather than re-running all 5 layers from scratch.

**Approach 2: Off-Chain Computation + ZKP Proof**

Move all 5-layer verification computation off-chain, generate a SNARK/STARK proof. On-chain only needs to verify Proof (~200k gas fixed cost, independent of delta count). This aligns with the ERC-8126 PDV (Private Data Verification) philosophy.

**Approach 3: Phased Deployment**

```
Phase 1 (MVP): Implement only L1 + L2 (quantity + pairing), gas ~80k
Phase 2: Add L3 (value conservation), extra ~30k
Phase 3: Add L4 (sequence check), extra ~40k
Phase 4: Add L5 (dual-world coupling), depends on oracle maturity
```

---

## IX. Security Analysis and Attack Surface

### 9.1 Attack Vector Matrix

| Attack Vector | Layer Affected | Defense | Residual Risk |
|---------|---------|---------|---------|
| **Fake dNFT creation** (forged +dNFT/-dNFT pair) | L1/L2 | DNFTRegistry only accepts writes authorized by DynamicNFT contract; createPair requires RWA binding proof | Low (requires compromising DynamicNFT) |
| **Value manipulation** (+dNFT value $1000, -dNFT value $10) | L3 | Value locked by oracle quote at creation, immutable thereafter; Layer 3 tolerance check | Low |
| **Skip-step attack** (directly CREATE→ANNIHILATE) | L4 | Layer 4 vertex sequence mandatory check; reject if intermediate states missing | Low |
| **Oracle malfeasance** (forged logistics signatures) | L5 | Multi-oracle redundancy (≥3 independent sources); Staking penalty mechanism | Medium (depends on oracle network quality) |
| **Evaluator contract itself replaced** | Global | ERC-8183 Evaluator address locked at Job creation, immutable; Proxy upgrade requires Timelock | Low |
| **Front-running evaluate()** | Timing | evaluate() result is deterministic (same input → same output); no MEV opportunity | None |
| **DoS (malicious mass delta generation)** | Gas | Delta count cap (e.g., single Job ≤ 50 dNFTs); reject beyond limit | Low |

### 9.2 Risk Overlay with ERC-8183 Inherent Risks

ERC-8183 security clause explicitly warns: "**malicious evaluator can arbitrarily complete or reject**."

FeynmanEvaluator **eliminates the possibility of subjective malfeasance** (decisions are determined by mathematics), but introduces new risks:

| New Risk | Probability | Impact | Mitigation |
|--------|------|------|------|
| Conservation algorithm bug causes legitimate transactions to be rejected | Low | High (funds locked) | Formal verification + multi-audit + TimeLock emergency override |
| Conservation algorithm vulnerability causes illegitimate transactions to be completed | Very Low | Critical (funds incorrectly released) | Section 11.7 insurance pool staking covers losses |
| Gas surge causes evaluate() to always OOM | Medium | Medium (Job stuck at Submitted) | Approach 1 incremental verification + fallback to manual arbitration |

**The most important mitigation is the Section 11.7 insurance pool/staking mechanism**: even if Evaluator mathematical logic is breached, the economic layer still has collateral covering losses.

---

## X. Implementation Roadmap (Phasing)

```
Phase 0 (Infrastructure, prerequisites)
  ├── DynamicNFT.sol refinement (pairId/chirality/status machine)
  ├── DNFTRegistry.sol development & deployment
  ├── JobBridge.sol development & deployment
  └── ERC-8183 Draft implementation (lock interfaces after Final)

Phase 1 (MVP: Minimum Runnable Evaluator)
  ├── FeynmanEvaluator v0.1: L1+L2 only (quantity + pairing)
  ├── deliverable format v0.1 (simplified, no ZKP)
  ├── Manual trigger mode (not Hook auto)
  └── Goal: correctly distinguish "normal trade pass / obvious anomaly reject"

Phase 2 (Production-Ready)
  ├── Add L3 (value conservation)
  ├── Add L4 (vertex sequence)
  ├── deliverable format v1.0 (with oracle signatures)
  ├── Hook auto-trigger integration
  └── Gas benchmarking & optimization

Phase 3 (Full Version)
  ├── Add L5 (dual-world coupling + oracle signature verification)
  ├── ZKP proof mode (optional alternative to full on-chain computation)
  ├── Section 11.7 insurance pool staking integration
  └── Formal verification (Certora/K)

Phase 4 (Ecosystem Expansion)
  ├── Multi-chain deployment (Base/Arbitrum/Optimism L2)
  ├── ERC-8004 reputation write-back integration
  └── Kleros arbitration as L5 fallback
```

---

## XI. Open Questions (To Be Researched/Decided)

> **✅ Confirmed Design Decision (moved out of open questions)**:
> Pure digital goods are **exempt from L5** (dual-world coupling layer). Digital goods (NFTs, e-books, API credentials, software licenses, on-chain credentials, etc.) have delivery that is intrinsically verifiable on-chain — via contract execution results, file hash matching, or buyer digital signature confirmation — and do not require physical-world RWA event anchors.
> L5 is enforced **only for transactions involving physical RWA**. This allows digital goods transactions to run evaluate() with just L1–L4, with lower gas costs and faster deployment, fully aligned with ERC-8183's design of allowing "contract Evaluator to handle deterministic tasks."
>
> | Verification Layer | Physical RWA Transaction | Pure Digital Goods Transaction |
> | :---: | :---: | :---: |
> | L1 Quantity Conservation | ✅ | ✅ |
> | L2 Pairing Integrity | ✅ | ✅ |
> | L3 Value Conservation | ✅ | ✅ |
> | L4 Vertex Sequence | ✅ | ✅ |
> | **L5 Dual-World Coupling** | **✅ Oracle anchors required** | **⚡ Exempt** |

| # | Question | Impact | Recommendation |
|---|------|------|------|
| 1 | **How to handle conservation under partial returns?** | Returning 1 of 3 items — how are dNFTs split? | May need "dNFT fractionalization" mechanism or "sub-pairing" concept |
| 2 | **Multi-merchant combined orders (one Job with dNFTs from multiple merchants)** | Is conservation scope per-job or per-merchant? | Per-job (a Feynman diagram can have multiple incoming/outgoing particles) |
| 3 | **How to collect δ in cross-chain scenarios?** | Goods on one chain, payment on another | Use cross-chain messaging (CCIP/LayerZero) to sync delta; or restrict to single-chain closure |
| 4 | **Should conservationHolds run on EVM or off-chain?** | Determines security model and gas costs | Phase 1-3 on-chain; Phase 4 optional ZKP off-chain |

---

## XII. Connectivity with Other Whitepaper Sections

```
Chapter 3 (Formal Framework) ──→ Provides mathematical definition of Σ(dNFT)=0
    │
    ▼
Appendix A (Mathematical Formalization) ──→ Provides formal description of vertex rules and propagator functions
    │
    ▼
Chapter 7 (Smart Contract Architecture) ──→ DynamicNFT.sol / Escrow.sol are data providers for this contract
    │
    ▼
Section 11.7 (Insurance Pool) ──→ Provides staking economic backing for Evaluator
    │
    ▼
Section 11.8 (This document's subject) ──→ FeynmanEvaluator: turns mathematical definitions into executable code
    │
    ▼
ERC-8183 Standard ──→ Defines the interface contract for evaluate/complete/reject
```

---

*Document Version: v1.0 | Date: 2026-07-12 | Status: Technical Analysis Draft (not final implementation)*
