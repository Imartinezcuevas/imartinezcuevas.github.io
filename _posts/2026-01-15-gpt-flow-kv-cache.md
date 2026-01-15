---
title: "How GPTs processes tokens and KV Cache"
description: "How information flows inside GPT during training and inference, explaining why autoregressive generation becomes expensive and how KV cache is used as an optimization."
author: imartinezcuevas
date: 2026-01-15 13:09:00 +0800
categories: [NanoLLM]
tags: [GPT, Deep Learning, Transformer, ML]
math: False
---

When I started this project, I knew that the model generated text token by token and couldn't see the future. But I couldn't explain how information flows inside the model, nor why inference becomes so costly as the context grows.

In this post, I want to open that black box: explain what happens inside a GPT when it processes a sequence, and what's the KV cache.

## From tokens to embeddings
Everything starts in a fairly straightforward way. Each token is transformed into a vector embedding that represents its meaning, and positional information is added on top of that. This embedding is not "intelligent" yet, it's simply the way a token enters the model.

From there, the vector passes through a stack of Transformer blocks. In real GPTs, this stack can have 12, 24, or even more blocks. All of them with the same structure, and each one progressively refines the token representations.

## What does a Transformer block really do?
Each transformer block can be seen as a black box that takes an embedding and returns a more contextualized version of it. Internally, this box has three key components:

### Multi-head attention
Attention is the mechanism that connects the present with the past. For each token, three projections are computed: Q(Query), K(Key) and V(Value). The query of the current token is compared with the keys of the previous tokens, and these comparisons determine wich values are used and with what weight.

The causal mask, as explained in the [previous post](https://proceedings.neurips.cc/paper_files/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf), guarantees that each position can only attend to previous tokens. However, it doesn't introduce any kind of memory. Attentions is purely an algebraic operation, it doesn't remember anything by itself. **This would be very important later on.**

### MLP layer
After attention, the embedding passes through a feed-forward network. There is not interaction between tokens here, each token is processed independetly. This layers allows the model to transform and amplify the information selected by attention.

An important detail is that the cost of this layers doesn't depend on the context lenght. Each token always pays the same cost when going through the MLP.

### Normalization and residual connections
Residual connections and normalization ensure that information doesn't degrade as it passes through many blocks. This is essential for the information flow to remain stable and trainable.

## Traning and inference
### Training: parallelism
During training, the model receives full sequences. With a single forward pass and causal masking, the model learns to predict each token using only its past. There is no step by step generaion and no temporal loop.

Each token computes its Q, K and V exactly one. There is no recomputation of past tokens, and therefore no need for any kind of cache.

### Inference: the autoregressive loop
During inference, the situation change completely. The sequence is incomplete: the model generates a token, append it to the context and runs the forward pass again. The forward is stateless function, so it doesn't remember anything between steps.

This means that, without optimization, at every step the model processes the entire previous sequence across all blocks, recomputing Q, K and V for token that were already processed. The cost grows fast as the context length increases.

## Why the KV cache is a game changer
The KV Cache appears as a natural consequence of the inference loop. If past tokens don't change, then their keys and values don't change either. Storing them allows reuse in subsequent inference steps.

With KV cache, at each step:
* The new token computes its Q, K and V.
* The keys and values of previous tokens are retrieved from the cache.
* Attention is computed using this past.

The model itself remains the same. The forward pass is conceptually identical. The only difference is that the past is no longer recomputed and instead is treated as a external state.