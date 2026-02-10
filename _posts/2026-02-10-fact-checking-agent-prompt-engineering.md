---
title: "I benchmarked 6 prompt engineering strategies to fix hallucinations"
description: "A deep dive into Zero-Shot, CoT, Self-Consistency, Decomposition, RAG, and ReAct. Trying to force Llama 3 (8B) to tell the truth."
author: imartinezcuevas
date: 2025-02-10 09:27:00 +0800
categories: [Prompt Engineering]
tags: [LLM, Prompt Engineering, RAG, Agents, Hallucinations, Fact Checking, NLP, Python]
---

I set up a fact-checking pipeline using the FEVER dataset (Fact Extraction and VERification) and a local instance of Llama 3.1 (8B). My goal was to verify objective claims (e.g., "Real Madrid won the 2024 Champion League").

I implemented and tested the six most popular prompt engineering paradigms:
1. Zero-Shot
2. Chain-of-Thought (CoT)
3. Self-Consistency
4. Decomposition
5. Retrieval augmented generation (RAG)
6. ReAct Agents

## Findings
Here is everything I learned about the limits of prompt engineering.

### 1. Zero-shot prompting
This is the baseline. You give the model a claim and ask for a verdict. No context, no examples, no "thinking".

*Prompt: "Claim: [X]. Verify if this supports, refutes or not enough info.*

#### The problems encountered:
* The model focuses on probability, not truth. If I asked about a fake event that sounded possible, the model often marked as supports simply because some words appear together in its training data.
* It rarely used the label *NOT ENOUG INFO*. It preferred to hallucinate a definite answer than admit ignorance.

Zero-shot proved that you can't use an LLM as a database.

### 2. Chain-of-Thought
Introduced by Google, CoT asks the model to generate a reasoning trace before answering.

*Prompt: "Verify the claim. Think step-by-step and explain your reasoning.*

I expected a jump in accuracy. I got a jump in "coherence", but not in correctness.

#### The problems encountered
* The model became better at math and logic, but for simple lookups facts, CoT just output more tokens to justify its own hallucinations.

CoT improves reasoning reliability, not fact retrieval. It helps the model connect A to B, but if it remembers A incorrectly, the chain of though just reinforces the error.

### 3. Self-Consistency
Instead of asking once, we sample the CoT prompt 5 times at a higher temperature and take the majority vote. The theory is that hallucinations are random, while truth is consistent.

#### The problems encountered
* The benchmark times skyrocketed. Verifying 50 claims took 5x longer. This is not factible for a real-world scenario. It will also triple the costs.
* LLM have biases. If the training data heavily associates entity X with Y, the model will hallucinate the same answer 5 times in a row.

Self-consistency filters out random noise, but it can't fix systematic erros in the model's weights. It's a "denoising" technique, not a "fact-checking" one.

### 4. Decomposition
For complex claims, we break them down into atomic facts. This was the hardest technique to implement.

#### The problems encountered
* The window grows significantly. By the time the model answers Q1 and Q2, it has forgotten the original instruction or the format it was supposed to follow.
* Sometimes the model would decompose a question into the same question rephrased. This added latency without adding value.

### 5. RAG
This is the industry standard. We search a database (or the web), retrieve the top-k chunks, paste them into the prompt, ans ask the model to verify.
RAG solved the "knowledge cutoff" problem, but introduced a new one: distractions.

#### The problems encountered
* Even with LLama 3.1 context window, if the relevant fact was buried in the middle of 5 irrelevant search results, the model often ignored.
* My pipeline used a basic search. Often, the top result was SEO-spam or a irrelevant Wikipedia page. The model would confidently anser based on irrelevant text, leading to false negatives.
* RAG is "one-shot". If the search query was slightly off, the retireval failed, and had no way to ask for a second try.

RAG is as good as your retrieval algorithm. If I had to do it for another project, I will expend more time on the search, implementing re-ranking and noise filtering.

### 6. ReAct Agent
The ReAct (Reasoning + Acting) allows the model to "speak" to the python environment. It generates a "Thought", then an "Action" (search), waits for an "Observation", and loops.

This was the most promising but also the most frustrating technique. It turned a text generation problem into a control systems problem.

#### Problems encountered
* Llama 3.1 is *good*, but not that *good*. After a few steps of reasoning, it would often forget the strict formatting rules I gave.
* The agent would sometime get stuck in an infinete loop. The model lacked the "meta-cognition" to realize its strategy wasn't working. I had to code limit iterations to stop these processes.
* Sometimes the search result contained text that looked like instructions. The model would read the observation and get confused, thinking it was the user giving a new command.

This gives the model agency, but requires constraint.

## What I learned about prompt engineering?
This project started as a quest to build and test different prompt strategies, but ended as a lesson in the limitations of AI.
1. There is no "God prompt": no combination of words will make an 8N model perfom like a better model.
2. Technique like Self-Consistency or ReAct improve the results but are way too expensive. A 20-second wai time for a simple fact-check is unacceptable.
3. Code > prompts. I improved the performance more by writing better python code and better retrievers than I did by tweaking the prompt text.

We are clearly not yet at the point where we can trust an 8B model to blindly navegate the web an tell us the truth. Hallucinations are a feature, not a bug, as they are a byproduct of the model's creativity. Constraining that creativity into rigid fact-checking requires more that clever prompts.