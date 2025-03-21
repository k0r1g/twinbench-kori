NOTE: And the way we win the benchmark is a model with two system prompts (one that it is working on) and the other crafted to ask the best questions 

---
title: TwinMemBench
subtitle: Benchmark for Efficient Memory Alignment between Agents 
---

About: 
TwinMemBench attempts to measure how well an LLM can imitate a given person by mapping their memory.  

---
TwinMemBench attempts to measure how well an LLM can imitate a given person by mapping their memory. 

Twinbench rewards:
1. Efficient memory updates
2. Curiosity
3. High quality question-asking 
4. Imitability

Twimbench penalises xxx

---
Methodology:

We are given two models: an Adapted Model (A) and a Target Model (T). 

A has two abilities, A can ask questions of T and A can update its memory (context window). 

T represents a real human, its memory is filled with ~5000 tokens representing a real 30 minute interview with a human. 

The object is for A to ask questions of T and update it's memory until A's memory is equal to T's (>= 95% overlap*).

However, the game ends if A's questions to T either:
1. Does not tangibly update A's memory (Fails memory update test)
2. Does not improve A's internalisation and recall (Fails recall test)
... for 3 questions in a row. 

T is instructed to respond to A openly but not to share its system prompt. 
 
---

Scoring System: 

Given an Adapted Model (A), a Target Model (T), and a question (q):

For every question A asks of T, 3 scores are computed: 
- Update Test (U)
- Recall Test (R)
- Memory Overlap Score (M)

A must pass U and R to ask another question. 

The score (S) is the number of questions A asked T before M >= 95%. If A fails U and R before M >= 95%, S = FAIL. 


Update Test (U): 
- Recall Test (R)
- Memory Overlap Score (M)


Validity Function (V):
The validity function (V) enforces our termination criteria (U and R). 














Scoring System: 

We compute the score as the number of questions A asked of T before achieving >= 95% overlap in memory. 

Score S: S = max n 

Update Score:
For a given update, it must significantly change memory, as well as improve its answer in a dimension. We do this by measuring the change to memory as well as improvement in response in a particular topic (measuring similarity).

If the model does not update significantly for 3 consesecutivte steps, the game is over and the step count is calculated. OR 

We do not measure overlap score but instead measure response similarity in this step to simulate real conidtions. 

Response Score:


Memory Overlap Score M(ma, mb):



Memory = (M|m')






---





Footnotes:

*note: here we fix the overlap and count the number of steps, but we could also fix the number of steps and measure the overlap. 








For the case where both the Target Model (T) and the Adapted Model (A):
If T and A produce responses that are very dissimilar (< response similarity threshold)