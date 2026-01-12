---
title: "Why GPTs are decoder-only"
description: "I share my learning process understanding why GPTs are decoder-only, focusing on outoregressive modeling and the importance of causality."
author: imartinezcuevas
date: 2026-01-12 11:39:00 +0800
categories: [NanoLLM]
tags: [GPT, Deep Learning, Transformer, ML]
---

Many people read ["Attention is all you need"](https://proceedings.neurips.cc/paper_files/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf) and assume that modern GPT models follow the same encoder-decoder architecture. They don't.

I made this assumption myself for a long time. At university, the learning path is usually something like this: perceptrons and basic neural networks, then sequence-to-sequence, and suddenly GPTs (often through libraries like Hugging Face). At no point you're encouraged to question whether the architecture itself has changed.

**So you naturally assume it hasn't.**

GPT models are built around a decoder-only architecture, designed specifically for autoregressive language modeling and next-token prediction. Understanding why this works requires shifting focus away from the original Transformer paper and toward the constraints imposed by the language modeling objective itself.

## What problem is a language model actually solving?
It's easy to think of a language model as something that "generates text". Sentences, paragraphs, answers. That intuiton feels natural, but it's fundamentally wrong.

A language model isn't defined by its architecture, but by the problem it's trained to solve. That problem is to model a probability distribution over sequences of tokens. In other words, the model is trained to answer a single question, over and over again: *Given everything I have seen so far, what's the most likely token?*

This has important consequences:
1. The model operates on a single sequence. There isn't a strict separation between "input" and "output" as in classical seq-to-seq models. What we usually call the input is just the beginning of a sequence, and what we call the output is its continuation.
2. The token order is fundamental. Each prediction is conditioned on all previous tokens. The model isn't learning to map one sequence to another but  to move forward in time, one step at a time.

From this point of view, the model never generates a chunk of text in one shot. It learns a process: observe context, predict next token, append it to the context, and repeat. Text generation is an emergent behavior of this iterative process, not a direct objective.

Once you understand this, the usual encoder-decoder framing starts to feel unnecessary. Encoder-decoder model makes sense when there's a clear boundary between input and output sequences, such as translation or summarization. But for pure language modeling, that boundary doesn't exist. There's only a growing context.

The constraints imposed by next-token prediction are strict:
* The model must be causal.
* It must not access future tokens.
* It must learn the conditional distribution $P(x_t \| x_{<t})$, not an easier version that leaks future information during training.

With this on the table, the architecture choices stop being arbitrary. Causal masking, autoregressive factorization, and the absence of an encoder aren't optimizations or simplifications, they're consequences.

## Why causality is not optional?
At training time, we already have access to the full sequence. Every token is known in advance. So a natural question arises:
*If the whole sentence is available, why not let the model use it?*

The answer is simple but fundamental: because that would change the problem.

The goal of a language model is to learn the conditional distribution $P(x_t \| x_{<t})$. If the model is allowed to see future tokens during training, it will learn a different and easier distribution $P(x_t \| x_{<t}, x_{>t})$. This may produce lower training loss, but it breaks the model.

During inference, future tokens don't exist. The model must generate one token at a time, relying only on past context. If training allows access to future information, the model learns dependencies it will never be able to use in production.

Causal masking is what enforces this constraint. It prevents each token from attending to tokens that come after it, forcing the model to operate under the same conditions during training and inference. The model can't "cheat" by looking ahead.