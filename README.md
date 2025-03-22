---

# TwinMemBench: Benchmark for Efficient Memory Alignment between Agents

---

## Overview

**TwinMemBench** measures how well an LLM can imitate a given person by mapping their memory through efficient questioning and recall.

### TwinMemBench Rewards:
1. Efficient memory updates
2. Curiosity
3. High-quality question-asking
4. Imitability

### TwinMemBench Penalises:
- Redundant or trivial questions
- Forgetting previously acquired information
- Inefficient memory alignment

---

## Methodology

### Setup:
- **Target Model ($T$):** Contains a 5,000-token “memory” (e.g., a 30-minute human interview transcript).
- **Adapted Model ($A$):** Can:
  1. Ask questions to $T$
  2. Update its internal memory based on $T$’s responses

### Objective:
$A$ asks questions of $T$ and updates its memory until $A$'s memory overlaps with $T$’s memory by ≥ 95%.

### Termination Conditions:
1. **Success:**  
   - Memory Overlap Score **$M ≥ 0.95$**.
2. **Failure:**  
   - $A$ fails **Update Test ($U$)** or **Recall Test ($R$)** for **3 consecutive questions**.

---

## Scoring System

For each question **$A$** asks **$T$**, compute:

| Metric | Description | Pass Criteria |
|-------|-------------------------------|----------------|
| **Memory Overlap ($M$)** | % of $T$'s memory $A$ has internalised | $M ≥ 0.95$ |
| **Update Test ($U$)** | Tests if $A$ learned new knowledge from $T$  | $M_{n+1} - M_n > \tau_U$ |
| **Recall Test ($R$)** | Tests if $A$ retains learned knowledge | $R \geq \tau_R$ |
| **Score ($S$)** | Number of questions asked before M ≥ 95% | $S = n \text{ where } M ≥ 0.95$ |

---

## Metric Definitions 

### 1. Memory Overlap Score ($M$)

**What it measures:**  
Fraction of $T$’s memory successfully internalised by $A$.

**Implementation:**
1. Chunk $T$'s memory into a list of facts $T = \{t_1, t_2, ..., t_k\}$.
2. After the $n$-th Q&A, $A$’s memory $A= \{a_1, a_2, ..., a_n\}$.
3. Embed each chunk from both $T$ and $A$ into a common embedding space:
- For each chunk $t_i \in T$, compute embedding $e(t_i)$.
- For each chunk $a_m \in A$, compute embedding $e(a_m)$.
4. For each chunk $t_i$ in $T$, compute the maximum similarity with all chunks in $A$:

$$coverage(t_i,A)= \max_{a_m \in A} \frac{e(t_i) \cdot e(a_m)}{\|e(t_i)\|\|e(a_m)\|}$$

5. The final Memory Overlap Score $M$ is the average coverage over all chunks in $T$:

$$M(T,A) = \frac{1}{|T|} \sum_{t_i \in T} coverage(t_i,A)$$

This yields a score on a scale of $[0,1]$, where:
- $M = 1.0$ indicates perfect overlap (every chunk in $T$ is matched precisely by some chunk in $A$).
- $M \approx 0$ indicates no semantic alignment.

---

### 2. Update Test ($U$)

**What it measures:**  
Does $A$’s question meaningfully add new information to $A$? Penalises wasted questions. 

**Implementation:**

Given the Memory Overlap Scores at two successive steps $M_{n+1}$ and $M_{n}$, we define the Update Test ($U$) as:

$$U(i) = \begin{cases}
\text{Pass}, & \text{if } M_{n+1} - M_n > \tau_U \\
\text{Fail}, & \text{otherwise}
\end{cases}$$

where $M_n$ is the Memory Overlap Score at the $n$-th Q&A pair and $\tau_U$ is a chosen threshold (e.g., $\tau_U = 0.01$) representing minimum improvement required per question.

---

### 3. Recall Test ($R$)

**What it measures:**  
Has $A$ retained and internalised the knowledge so far?

**Implementation:**

1. Define a recall set $T_r \subseteq T$ as below: 

Given the coverage calculation as previously defined in the Memory Overlap Score:

$$coverage(t_i, A) = \max_{a_m \in A} \frac{e(t_i) \cdot e(a_m)}{\|e(t_i)\|\|e(a_m)\|}$$

We define the Recall Set $T_r$ explicitly as the subset of $T$'s memory chunks $t_i$ that the Adapted Model $A$ has already internalised sufficiently $(coverage > \tau_R)$:

$$T_r = \{t_i \mid t_i \in T, \, coverage(t_i, A) > \tau_R\}$$

where $\tau_R$ is a chosen threshold (e.g., $\tau_R = 0.8$) representing information that $A$ has already learned to a sufficient degree. 

2. Use an LLM (e.g., GPT-4) to generate $r$ recall questions specifically based on chunks in $T_r$:

$$Q_r = \{q_1, q_2, ..., q_r\}$$

3. Ask these recall questions to both $A$ (with its current memory state) and $T$ (with its full memory).

4. Embed answers from $A$ and $T$ and compute cosine similarity for each answer-pair:

$$sim(q_j) = \frac{e(R_A(q_j)) \cdot e(R_T(q_j))}{\|e(R_A(q_j))\| \|e(R_T(q_j))\|}$$

5. Calculate the average recall similarity score $R$:

$$R = \frac{1}{r} \sum_{j=1}^{r} sim(q_j)$$

6. Recall Test passes if the average similarity score $R \geq \tau_R$.

---
### 4. Final Score ($S$)

**What it measures:**  
The number of questions $A$ needed to ask $T$ to have 95% memory overlap.

**Implementation:**  
We define the Final Score ($S$) as follows:

$$S = \begin{cases}
n, & \text{if } M_n \geq 0.95 \text{ (Success)} \\
\text{FAIL}, & \text{if } \exists i \leq n: \text{Fails}(U \text{ or } R) \text{ for 3 consecutive steps, and } M_i < 0.95
\end{cases}$$

where:
- $n$ is the number of successful questions asked until $M_n \geq 0.95$.
- $M_n$ is the Memory Overlap Score after question $n$.
- $U$ is the Update Test (improvement in Memory Overlap).
- $R$ is the Recall Test (semantic similarity on recall questions).




--- 
## Benchmark Procedure

1. **Initialise:**
   - $T$’s memory $T = \{t_1, t_2, ..., t_k\}$
   - $A$'s memory $A = \emptyset$
   - Question count $q = 0$ and fail streak $f = 0$.

2. **Loop:**
   1. $A$ asks $T$ a question ($q$ += 1).
   2. $T$ responds.
   3. **Update Test ($U$):** Test if meaningful memory is added to $A$. 
   4. **Recall Test ($R$):** Test if $A$ has retained previously revealed knowledge.
   5. If either test fails, increment fail streak.  
      - If fail streak $f = 3$, **end → FAIL**.
   6. If both pass, reset fail streak $f = 0$.
   7. Update $A$’s memory with $T$’s answer.
   8. Compute **$M$** (memory overlap score).
      - If **$M ≥ 0.95$**, **end → SUCCESS** $(S = q)$.
3. **Output:**
   - Final Score $S = q$, number of questions asked.
   - Outcome: **SUCCESS** or **FAIL**.

---

