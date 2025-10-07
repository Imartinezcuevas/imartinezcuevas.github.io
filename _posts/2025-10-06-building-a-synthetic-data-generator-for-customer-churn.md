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
#### `plan_type`
We assign weighted probabilities to each plan type that evolve over time:
* **Early stage (e.g., 2024)** --> mostly free user.
* **Later stage (e.g., 2025)** --> more premium adoption.
We can either interpolate gradually to simulate gradual drift or switch abruptly for sudden drift. This helps test how the pipeline reacts to changing user composition.

#### `montly_spend`
Spend depends on plan type:
* **Free:** always 0.
* **Basic:** Normal(mean=20, std=5)
* **Premium:** Normal(mean=50, std=10)
I've also add seasonality, e.g., spending spikes during December, to simulate realistic temporal behaviour.

#### `support_tickets`
Number of support interactions can indicate dissatisfaction. We simulate it with a Poisson distribution, conditional on plan type:

| Plan    | $\lambda$ |
| :------ | :-------- |
| Free    |    1.5    |
| Basic   |    1.0    |
| Premium |    0.5    |

Higher $\lambda$ --> more interactions --> higher churn probability. This featrue helps demonstrate conditional relationships.

#### `region`
We simulate 4 regions with different churn tendencies:
* **NA, EU, LATAM, APAC**
Each region has a multiplier applied to churn probability, showing how geographic context influences user behaviour.

#### `device_type`
We assign each user a device type:
* **Mobile or desktop**
The device affects session length and slightly modifies churn propablity.

#### `avg_session_length`
Depends on plan type and device type:
| Plan    |        Mobile          |        Desktop         |
| :------ | :--------------------- | :--------------------- |
| Free    | Normal(mean=5, std=2)  | Normal(mean=7, std=3)  |
| Basic   | Normal(mean=10, std=3) | Normal(mean=12, std=4) |
| Premium | Normal(mean=15, std=5) | Normal(mean=20, std=5) |

#### `churned`
The `churned` columns is the target variable, so its generation is crucial for realism. We model it through a logistic function that combines multiple behavioural and contextual signals. This mimics how churn behaves
in the real world, where several weak factors combine to push customers toward staying or leaving.

## Conclusion
The next step is to plug the generator into a streaming infrastructure, so it behaves like a real system rather than a static dataset. A practical choice is to use Kaftka as the message broker.

Additionally, we can integrate it with orchestration tools like Airflow to automate daily generation, drift injection, and retraining triggers. And use monitoring dashboards to observe the synthetic system in real time.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The complete code for this development is avaliable [**here**](https://github.com/Imartinezcuevas/ml-churn-simulation/blob/main/data-generation/generator.py).
{: .prompt-tip }
<!-- markdownlint-restore -->

