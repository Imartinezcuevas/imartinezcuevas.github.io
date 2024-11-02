---
title: "Linear Regression"
date: "2024-11-02"
summary: "A comprehensive introduction to linear regression, exploring its fundamental concepts and mathematical foundations."
description: "An in-depth guide to linear regression, covering basic principles and key concepts to build a strong foundational understanding of this essential machine learning algorithm."
toc: true
readTime: true
autonumber: true
math: true
tags: ["Linear Regression", "Python", "Machine Learning", "AI", "Artificial Intelligence"]
showTags: false
---

## What is Linear Regression?
Linear regression is one of the most important and widely used regression techniques. It is a statistical method used to model relationships between variables, specifically when there is a linear relationship between them.

### A Closer Look
Linear regression is a supervised algorithm that learns to model a dependent variable, $ y $, as a function of some independent variables (often called 'features'), $ x_i $, by identifying the line or surface that best fits the data. In general, the equation for linear regression is:

$$ y = b + w_1 \cdot x_1 + w_2 \cdot x_2 + \dots + w_i \cdot x_i $$

where:
* $ y $: the dependent variable, or what we aim to predict.
* $ x_i $: the independent variables, or the features the model uses to predict $ y $.
* $ b $: the y-intercept of the regression line.
* $ w_i $: the weights or coefficients the model learns during optimization.

The simplest linear regression model can be represented graphically as a best-fit line between data points, while a multiple linear regression model can be represented as a plane (in 2 dimensions) or a hyperplane (in higher dimensions). Despite their differences, both simple and multiple regression models are linear, as they take the form of a linear equation.

## Training
Training a linear regression model involves finding the set of weights that best predict $ y $ based on the features. Although we may not know the true parameters, we can estimate them. Once these coefficients, $ \hat{w}_i $, are estimated, we can predict future values, $ \hat{y} $, as follows:

$$
\hat{y} = \hat{w}_1 \cdot x_1 + \hat{w}_2 \cdot x_2 + \dots + \hat{w}_i \cdot x_i
$$

The process for estimating these parameters involves the following steps:
1. **Random weight initialization**: $ w_i $ is initially unknown, so we begin by setting random values for the weights or initializing them to 0.
2. **Generate predictions**: Use the initialized weights in the equation to predict outcomes for each observation.
3. **Calculate the loss**: Compute the Residual Sum of Squares (RSS), which is the sum of squared differences between the actual and predicted values:
   $$
   \text{RSS} = \sum_{i=1}^{n} (y_i - \hat{y}_i)^2
   $$
4. **Minimize the RSS**: The model finds the best parameters by defining a cost function and minimizing it through a method like gradient descent.

This process is repeated until the error reaches a minimum, and gradient descent cannot reduce the cost function further.

## Regularization
Although linear regression may seem unlikely to overfit, as it models data with a single line (or hyperplane in higher dimensions), overfitting can still occur, especially when features are correlated or redundant.

Consider two nearly identical features. This results in a predicted hyperplane of the form:
$$
\hat{y} = w_1 \cdot x_1 + w_2 \cdot x_2 + w_0
$$

If $ x_2 $ is nearly identical to $ x_1 $ (perhaps differing by a small multiplicative factor or some noise), both terms may be unnecessary. In such a case, a simpler model would be:
$$
\hat{y} = w_1 \cdot x_1 + w_0
$$

This simpler model is less complex and potentially more robust. 

The challenge is that identifying redundant features by inspection is often impractical. Ideally, the model would automatically prefer simpler representations when possible, as long as they fit the data adequately.

To achieve this, we add a penalty term to the loss function. Two common options for these penalties are the L2 norm (Ridge regression) and the L1 norm (Lasso regression):

* **L2 norm**: Adds the square of each weight to the loss function, discouraging large weights.
* **L1 norm**: Adds the absolute values of the weights to the loss function, promoting sparsity by pushing some weights to zero.

These penalties help the model avoid overfitting by favoring simpler, more generalizable solutions.

### L2 and L1 Penalties in Linear Regression

#### No Regularization (Standard Linear Regression)

This is the basic linear regression model without any added regularization:

$$
L(w) = \sum_{i=1}^{n} (y_i - w \cdot x_i)^2
$$

#### L2 Regularization (Ridge Regression)

By adding an L2 penalty term to the loss function, we implement **L2 regularization**:

$$
L(w) = \sum_{i=1}^{n} (y_i - w \cdot x_i)^2 + \lambda \sum_{j=0}^{d} w_j^2
$$

The **L2 penalty** includes the L2-norm of $ w $, and the resulting loss function is known as **Ridge regression**.

##### Explanation

We aim to minimize the loss function, so any additional terms (like penalties) are also minimized. Adding a penalty term encourages smaller weights. The first term represents how well the model fits the data, while the second term reduces the magnitude of the weights.

The parameter $ \lambda $ (often denoted as $ \alpha $ in libraries like scikit-learn) adjusts the strength of the penalty:

- A very low $ \lambda $ value reduces the penalty effect, allowing for a more complex model.
- Setting $ \lambda = 0 $ results in the standard linear regression model.
- A high $ \lambda $ value simplifies the model, though it may reduce data fit quality.

##### Choosing $ \lambda $

Choosing the appropriate $ \lambda $ value often requires experimentation. We typically try values in an exponential range (e.g., 0.01, 0.1, 1, 10) and select the one that minimizes the loss on a validation set or through k-fold cross-validation.

#### L1 Regularization (Lasso Regression)

Alternatively, we can apply the L1-norm to achieve **L1 regularization**:

$$
L(w) = \sum_{i=1}^{n} (y_i - w \cdot x_i)^2 + \lambda \sum_{j=0}^{d} |w_j|
$$

##### Explanation

Lasso regression has a distinct effect: it promotes sparsity by setting some weights to zero, effectively performing feature selection. Unlike Ridge regression, however, it lacks a straightforward closed-form solution and is generally solved using optimization techniques.

## Limitations of Linear Regression

While linear regression is a powerful and widely-used technique, it does have some limitations:

1. **Linear Relationships Only**: Linear regression assumes a linear relationship between the dependent and independent variables. If the relationship is non-linear, linear regression may not capture it effectively, leading to poor predictions. In such cases, other algorithms like polynomial regression or neural networks may be more suitable.

2. **Sensitivity to Outliers**: Linear regression models are highly sensitive to outliers. A few extreme values can heavily influence the best-fit line, resulting in a model that does not generalize well to typical data points. Techniques like robust regression or removing outliers from the dataset can sometimes mitigate this.

3. **Multicollinearity**: When features are highly correlated, it can make the model unstable and challenging to interpret. Multicollinearity can lead to large swings in coefficient estimates with minor changes in the data. Ridge regression or principal component analysis (PCA) are often used to address multicollinearity.

4. **Overfitting in High Dimensions**: Linear regression can overfit when there are too many features relative to the number of observations. Regularization techniques like Lasso and Ridge regression help mitigate this, but in cases where the number of features is very large, it may be beneficial to use techniques like dimensionality reduction.

5. **Assumption of Normally Distributed Errors**: Linear regression assumes that residuals (errors) follow a normal distribution. If this assumption is violated, such as in the presence of skewed data, the model’s performance can degrade. In these cases, transformations of the dependent variable or using a non-linear model might improve results.