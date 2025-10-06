---
title: "Building a synthetic data generator for customer churn"
description: "Realistic synthetic data generation with drift and seasonality for ML pipeline testing."
author: imartinezcuevas
date: 2025-10-06 13:58:00 +0800
categories: [ML Customer Churn]
tags: [Python, MLOps, ML, Synthetic data]
---

In this article, I'm going to explain how I build a realistic synthetic data generator fora customer churn ML project. The generator can produce streaming data, simulate different kinds of data drift (gradual, sudden, seasonal)
, and be configured for testing ingestion pipelines, monitoring, and automated retrain.

## Data Schema
```json
{
  "customer_id": "uuid",                 # Unique identifier for each customer
  "signup_date": "YYYY-MM-DD",           # When the customer joined
  "last_active_date": "YYYY-MM-DD",      # Last interaction with the product
  "plan_type": "free|basic|premium",
  "monthly_spend": "float",                # Derived from plan type
  "support_tickets": "int",                # Number of support interactions
  "region": "categorical",               # Different churn rates by market
  "device_type": "categorical",          # Mobile vs desktop usage patterns
  "avg_session_length": "float",           # Average session duration
  "churned": "0|1"
}
```
This schema captures identity, behaviour, demographics, and outcome, which allows realistic simulation.

## Making it realistic
Realism comes from combining sensible marginal distribution and conditional dependencies.
### `plan_type`: categorical distribution with time drift
We assign weighted probabilities to each plan type that evolve over time:
* Early stage (e.g., 2024) --> mostly free user.
* Later stage (e.g., 2025) --> more premium adoption.
We can either interpolate gradually to simulate gradual drift or switch abruptly for sudden drift. This helps test how the pipeline reacts to changing user composition.

### `montly_spend`: conditional numeric distribution
Spend depends on plan type:
* Free: always 0.
* Basic: Normal(mean=20, std=5)
* Premium: Normal(mean=50, std=10)
I've also add seasonality, e.g., spending spikes during December, to simulate realistic temporal behaviour.

### `support_tickets`: count distribution
Number of support interactions can indicate dissatisfaction. We simulate it with a Poisson distribution, conditional on plan type:
* Free: 1.5\lambda
* Basic: 1.0\lambda
* Premium: 0.5\lambda
Higher \lambda --> more interactions --> higher churn probability. This 
