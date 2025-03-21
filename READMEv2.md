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

### TwinMemBench Penalizes:
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
$A$ asks questions of $T$ and updates its memory until **$A$'s memory overlaps with $T$’s memory by ≥ 95%**.

### Termination Conditions:
1. **Success:**  
   - Memory Overlap Score **$M ≥ 95%$**.
2. **Failure:**  
   - A fails **Update Test ($U$)** or **Recall Test ($R$)** for **3 consecutive questions**.

---

## Scoring System

For each question **$A$** asks **$T$**, compute:

| Metric | Purpose | Pass Criteria |
|-------|--------|----------------|
| **Update Test ($U$)** | Checks if $A$’s memory gained new knowledge | $A$'s memory covers new portion of T’s memory |
| **Recall Test ($R$)** | Ensures $A$ retains previously acquired info | $A$ passes pop-quiz on revealed knowledge |
| **Memory Overlap Score ($M$)** | Measures total alignment between A and T | ≥ 95% overlap → success |

**Score (S):**
- **S = Number of successful questions asked until M ≥ 95%.**
- **S = FAIL** if A fails U or R 3 times before reaching M ≥ 95%.

---

## Detailed Metric Definitions

### 1. Memory Overlap Score (M)

**What it measures:**  
Fraction of T’s memory chunks successfully internalized by A.

**Implementation:**
1. **Chunk** T’s memory into \( \{t_1, t_2, ..., t_K\} \) (e.g., paragraphs/facts).
2. After each Q&A, A’s memory = \( \{a_1, a_2, ..., a_j\} \).
3. For each \( t_i \), check if any \( a_m \) is semantically similar (cosine similarity ≥ threshold).
4. Compute:
   \[
   M = \frac{|\{t_i : \exists a_m \text{ with similarity} \geq \tau\}|}{K}
   \]
5. If **M ≥ 0.95**, A has achieved memory alignment.

---

### 2. Update Test (U)

**What it measures:**  
Does A’s question lead to **new knowledge**?

**Implementation:**
1. After T answers, check if any new chunk \( t_i \) is covered that wasn’t covered before.
2. **Pass:** If T’s response reveals previously unrevealed memory chunks.
3. **Fail:** If T’s response is trivial, repetitive, or yields no new memory coverage.

---

### 3. Recall Test (R)

**What it measures:**  
Has A retained and internalized the knowledge so far?

**Implementation:**
1. After each Q&A, quiz A on randomly selected previously revealed chunks:
   - Ask factual questions
   - Ask A to summarize parts of T’s memory
2. Compare A’s answers to T’s known memory (via embedding similarity or factual matching).
3. **Pass:** ≥ 90% accuracy on quiz.
4. **Fail:** Below accuracy threshold.

---

## Benchmark Procedure

1. **Initialize:**
   - T’s memory = \( \{t_1, ..., t_K\} \)
   - A starts empty memory.
   - Question count \( q = 0 \), fail streak = 0.

2. **Loop:**
   1. A asks T a question (\( q += 1 \)).
   2. T responds.
   3. **Update Test (U):** Check if new memory chunks are gained.
   4. **Recall Test (R):** Quiz A on past revealed knowledge.
   5. If either test fails, increment fail streak.  
      - If fail streak = 3, **end → FAIL**.
   6. If both pass, reset fail streak = 0.
   7. Update A’s memory with T’s answer.
   8. Compute **M** (memory overlap score).
      - If **M ≥ 95%**, **end → SUCCESS (S = q)**.
3. **Output:**
   - Final Score **S** = Number of successful questions asked.
   - Outcome: **SUCCESS** or **FAIL**.

---

## Example Walkthrough

1. **Start:**  
   - A’s memory = empty (0% overlap).
2. **Question 1:**  
   - A asks "Tell me about your childhood."
   - T replies with childhood info.
   - **Update Test:** Pass (new chunk gained).  
   - **Recall Test:** Pass (A recalls correctly).
   - **M:** Updated to 5%.
3. **Question 2:**  
   - A asks trivial repeat.
   - T rehashes same info.
   - **Update Test:** Fail (no new coverage).
   - **Recall Test:** Pass.
   - Fail streak = 1.
4. **Continue:**  
   - If fail streak hits 3 before 95% memory overlap, **FAIL**.
   - If coverage ≥ 95%, **SUCCESS**, score = number of questions.

---

## Optional Enhancements

- **Token Efficiency:**  
  Track total tokens used (questions + answers).
- **Hallucination Penalty:**  
  Penalize contradictory or false info introduced by A.
- **Partial Coverage Score:**  
  Track % coverage at point of failure.
- **Summarization Option:**  
  Allow A to summarize rather than verbatim reproduce, with relaxed matching.

---

## Summary Table

| Metric | Description | Pass Criteria |
|-------|-------------------------------|----------------|
| **Memory Overlap (M)** | % of T's memory A has internalized | ≥ 95% |
| **Update Test (U)** | Checks if new knowledge gained | New chunk covered |
| **Recall Test (R)** | Tests if A retains known info | ≥ 90% quiz accuracy |
| **Score (S)** | # of successful questions asked before M ≥ 95% | Lower = better |

---

**TwinMemBench** provides a rigorous, quantifiable measure of how efficiently and accurately an LLM can align its memory with a target agent by strategically asking questions and retaining knowledge.

---

Would you like me to format this into a proper GitHub README or paper draft structure too?