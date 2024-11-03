---
title: "K-Nearest Neighbors"
date: "2024-11-03"
summary: "An overview of the K-Nearest Neighbors (KNN) algorithm."
description: "A detailed exploration of the K-Nearest Neighbors algorithm, providing insights into its operational mechanics, mathematical underpinnings, and practical implementations, while also addressing its strengths and limitations."
toc: true
readTime: true
autonumber: true
math: true
tags: ["K-Nearest Neighbors", "KNN", "Python", "Machine Learning", "AI", "Artificial Intelligence"]
showTags: false
---

## What is K-Nearest Neighbors
The K-Nearest Neighbors (KNN) algorithm is a widely-used method in machine learning for classification and regression tasks. Based on the principle that similar data points tend to be located close to each other in feature space, KNN provides a simple yet powerful approach to making predictions.

### How KNN works
KNN is a supervised learning algorithm, which means it requires labeled training data to make predictions. The algorithm predicts the class or value of a new instance based on the classes or values of its $k$ nearest neighbors in the training dataset. The essence of KNN lies in measuring the distance between points, typically using Euclidean distance, although other distance metrics can also be employed.


## Mathematical Representation
For classification, let’s denote:

* $x$: the new instance to classify. 
* $X$: the training dataset.
* $i$: the class labels of the instances in $X$.
* $k$: the number of nearest neighbors.

The predicted class $\hat{y}$ for the new instance can be expressed as:
$$
\hat{y} = mode(y_{i_1}, y_{i_2}, \dots, y_{i_k})
$$

where $i_1, i_2, \dots, i_k$ are the indices of the $k$ nearest neighbors to $x$.

For regression, the predicted value 
$$
\hat{y} = 1/k \sum_{j=1}^{k} y_j
$$
where $y_j$ is the value of the $j$-th neighbor.

## Training
Unlike many other algorithms, KNN does not undergo a traditional training phase. Instead, it stores the entire training dataset and makes predictions on the fly using the data. This makes KNN a lazy learner, as it waits until a query is made to perform the computation.

The general process of KNN can be summarized as follows:

1. **Data Storage**: The training data is stored in memory.
2. **Distance Computation**: When a new instance $x$ is presented, the algorithm computes the distance from $x$ to all instances in the training set.
3. **Neighbor Identification**: The $k$ closest instances are identified based on the distance metric used.
4. **Voting/Averaging**:
    * For classification, the algorithm votes among the $k$ neighbors to determine the class label.
    * For regression, it averages the values of the $k$ neighbors to predict the output.

## Advantages of KNN
* **Simplicity**: KNN is easy to understand and implement, making it an ideal algorithm for beginners.
* **No Training Phase**: It is efficient for small datasets since there is no need for training, and predictions are made based on the stored data.
* **Versatility**: KNN can handle both classification and regression tasks.

## Disadvantages of KNN
* **Computationally Intensive**: KNN can be slow with large datasets since it computes distances to all training instances for each prediction.
* **Memory Usage**: The algorithm requires storage for the entire training dataset, which may be impractical for large datasets.
* **Sensitive to Noise**: KNN can be influenced by irrelevant features or noise in the data, especially with a small value of $k$.
* **Curse of Dimensionality**: The performance of KNN tends to degrade in high-dimensional spaces where distance measures become less meaningful.