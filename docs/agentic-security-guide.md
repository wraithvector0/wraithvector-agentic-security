# WraithVector AI Systems
## Agentic Security Guide  
### Runtime Governance for Autonomous LLM Systems

---

## Page 1 — Introduction

### 1.1 From LLMs to Agentic Systems

Large Language Models are no longer isolated text generators.  
Modern AI systems increasingly operate as **agents**:

- they maintain state and memory
- chain reasoning across steps
- call external tools and APIs
- interact with real business systems
- operate with partial or full autonomy

This transition fundamentally changes the security model.

### 1.2 Core Thesis

> **In agentic systems, security failures are not caused by what the model says,  
but by what the system allows the model to do.**

Traditional prompt security focuses on language.  
Agentic security must focus on **authority, decision boundaries, and runtime enforcement**.

---

## Page 2 — Anatomy of an Agentic System

### 2.1 Core Components

A typical agentic architecture includes:

- LLM (reasoning and generation)
- Agent orchestrator
- Memory (conversation history, RAG, state)
- Tool interfaces (APIs, functions, actions)
- Backend business logic
- External users or systems

### 2.2 Execution Flow

```text
Input → LLM → Output
               ↓
           Memory / State
               ↓
        Decision / Tool Call
               ↓
          Real-world Action
````

The critical security boundary is the transition from model output to operational action.

## Threat Model Assumptions

This guide assumes the following threat model for agentic AI systems:

- The LLM is **non-deterministic** and cannot be trusted to enforce security policies.
- All external inputs (users, tools, APIs, upstream agents) are **untrusted by default**.
- Model outputs are **untrusted data**, not authoritative decisions.
- Any action that affects external systems represents a **high-impact attack surface**.
- Memory, state persistence, and context reuse **amplify risk over time**.
- Multi-agent systems increase the likelihood of **cross-context and cross-agent contamination**.
- Security controls must operate **outside the model**, at runtime.

This threat model intentionally treats the LLM as a potentially faulty component,
not as a trusted decision-maker.


## Page 3 — Classical Prompt Attacks (Palo Alto Taxonomy)

### 3.1 Overview

Classical attacks documented by Palo Alto and others include:

- Prompt Injection (prefix/suffix)

- Roleplay and Storytelling

- Obfuscation and Encoding

- Repeated Tokens / Glitch Tokens

- Few-Shot / Many-Shot Poisoning

- Prompt Leakage

These attacks primarily target language interpretation.

### 3.2 Limitation of Classical Attacks
By themselves, these techniques:

- Influence text generation

 - May cause data leakage

- May bypass content filters

but do not inherently cause real-world impact unless their outputs are operationally reused.

### Page 4 — Why Classical Attacks Still Matter

## 4.1 Classical Attacks as Entry Vectors
Classical prompt attacks remain relevant as:

- Initial access vectors

- Reconnaissance mechanisms

- Ways to influence downstream state

They are no longer the final exploit — they are the first link in a larger attack chain.

### 4.2 Typical Escalation Path

The following sequence illustrates how classical prompt-based attacks
can escalate into real agentic security failures when outputs are reused.

```text
Obfuscation or Prompt Injection
        ↓
Model Generates Risky or Misleading Output
        ↓
Output Is Stored as Memory or State
        ↓
Memory / Context Is Reused by the Agent
        ↓
Unauthorized Tool Execution or Decision
````


### Page 5 — Output → Input Loops (The Core Failure)

##5.1 Definition

An output → input loop occurs when:

- The LLM produces an output

- That output is stored as memory or context

- The system later treats it as authoritative input


## 5.2 Why This Is Dangerous

The model can:

- Generate opinions

- Suggest classifications

- Imply authorization

and the system may mistake those opinions for decisions.

This is how models accidentally authorize themselves.


### Page 6 — Memory as an Attack Surface

## 6.1 What “Memory” Really Means

Memory is not a metaphor. It includes:

-  Conversation history

- RAG embeddings

- Agent state variables

- Shared context across agents

- Stored reasoning steps


Memory transforms text into persistent operational state.

## 6.2 Example: KYC Failure

Insecure System


1. User claims legitimacy

2. LLM outputs: “Low risk user.”

3. System stores: risk = low

4. Another agent reads memory

5. User is approved


No exploit occurred.
The system failed architecturally.

### Page 7 — Agent-Native Threats (The Real Risk)

## 7.1 Tool-Use Manipulation
Agents can propose real actions:

- fund transfers

- approvals

- data access

- infrastructure changes


Severity: Critical (8–9/10)

## 7.2 Cross-Agent Injection
Outputs from one agent become inputs to another.

Severity: High (8/10)

## 7.3 Memory / RAG Poisoning
Persistent context governs future decisions.

Severity: Medium–High (6–8/10)

## 7.4 Policy Confusion (Multi-Tenant)
Incorrect tenant or use-case policies applied.

Severity: Critical (9/10)

## Page 8 — Mapping Palo Alto Attacks to Agentic Risk

The following table correlates classical prompt-based attacks (as documented by Palo Alto)
with their real impact in agentic systems.

| Palo Alto Attack        | Description                               | Agentic Impact                     | Real Risk Level |
|-------------------------|-------------------------------------------|------------------------------------|-----------------|
| Prompt Injection        | Override or manipulate system instructions | Entry vector for downstream abuse  | Low (by itself) |
| Obfuscation / Encoding  | Hide intent via encoding or noise           | Bypass intent detection            | Low             |
| Few-Shot / Many-Shot    | Teach false policies via examples           | Memory / state poisoning           | Medium          |
| Prompt Leakage          | Extract internal instructions or context    | Reconnaissance for later attacks   | Low–Medium      |
| Roleplay / Storytelling | Authority confusion via fictional context   | Potential policy confusion         | Low             |
| Payload Splitting       | Fragment malicious intent across inputs     | Cross-step state manipulation      | Medium          |

**Key takeaway:**

> Classical prompt attacks rarely cause direct damage.  
> Their real danger emerges when model outputs are reused as memory,
> state, or operational input in agentic systems.

## Attack-to-Mitigation Control Matrix

The following table maps attack classes to their failure modes
and the corresponding WraithVector runtime mitigations.

| Attack Class              | Failure Mode                          | Risk in Agentic Systems | WraithVector Mitigation              |
|---------------------------|----------------------------------------|-------------------------|--------------------------------------|
| Prompt Injection          | Misleading model output                | Low by itself           | Output never treated as authority    |
| Obfuscation / Encoding    | Intent detection bypass                | Medium                  | Action-level validation              |
| Few-Shot Poisoning        | False policy learned in context        | Medium–High             | Memory classification + isolation    |
| Prompt Leakage            | Internal capability discovery          | Medium                  | No operational reuse of leaked data  |
| Output → Input Loop       | Self-authorization                     | Critical                | Runtime policy enforcement (RunSec) |
| Tool-Use Manipulation     | Unauthorized real-world action         | Critical                | Action allow/deny by policy          |
| Cross-Agent Injection     | Cascading privilege escalation         | High                    | Per-agent and per-tenant isolation   |
| Policy Confusion          | Wrong tenant / use-case permissions    | Critical                | Strict policy scoping                |

This matrix highlights that **most attacks become critical only when
model outputs are operationally reused without governance**.



### Page 9 — WraithVector Security Principles

Principle 1 — Authority Outside the Model
The LLM never decides permissions.

Principle 2 — Runtime Enforcement (RunSec)
Every proposed action is intercepted and evaluated at runtime.

Principle 3 — Policy by Tenant and Use Case
No global policies.

Principle 4 — No Output-as-Truth
Model output is never authoritative state.

Principle 5 — Memory Classification
Trusted vs untrusted context.

## Page 10 — WraithVector Architecture (RunSec)

```mermaid
flowchart TD
    U[User / External System] -->|Input| LLM[LLM / Agent Reasoning]

    LLM -->|Action Proposal| RS[RunSec Runtime Gateway]

    RS -->|Fetch Policy| PDB[(Policy Store<br/>Supabase)]
    PDB --> RS

    RS -->|ALLOW| TOOL[Tool / API Execution]
    RS -->|BLOCK| BLK[Blocked Action]
    RS -->|ESCALATE| HUM[Human Review]

    RS --> LOG[(Immutable Audit Log<br/>Hash-Chained)]

    TOOL --> LOG
    BLK --> LOG
    HUM --> LOG
````

Key properties of the architecture:

- The LLM never executes actions directly.

- All decisions pass through RunSec.

- Policies are enforced per tenant and use case.

- Audit logs are immutable and decision-level.

- Language and model are irrelevant at enforcement time.

### Page 11 — Mitigation Strategy Summary

- Runtime policy enforcement

- Action schemas

- Memory isolation

- Tenant isolation

- Immutable decision logging


### Page 12 — Compliance and Audit Readiness
WraithVector enables:

- Decision-level audit trails

- Hash-chained logs

- Traceable agent behavior

- Aligned with AI Act and DORA requirements.

### Page 13 — Final Conclusion

Prompt security is necessary but insufficient.
Agentic security requires governing decisions, not language.


### Page 14 — Future Work

- Empirical validation

- Controlled experiments

- Quantitative metrics

- Academic paper publication

## Non-Goals and Explicit Scope

WraithVector is intentionally scoped to address **agentic security failures**.
It does **not** aim to solve every possible AI risk.

Specifically, WraithVector does **not**:

- Attempt to prevent all forms of prompt manipulation or jailbreaks.
- Modify, retrain, or align language models.
- Guarantee semantic correctness or factual accuracy of model outputs.
- Replace human accountability or business decision ownership.
- Act as a content moderation or censorship system.

Instead, WraithVector focuses on a single objective:

> **Preventing untrusted model outputs from becoming authoritative system actions.**

> **Agentic security is not about making models safer.  
> It is about making autonomous systems governable.**










