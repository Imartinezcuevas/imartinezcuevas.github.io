---
title: "Building a Reddit sentiment analysis platform"
description: ""
author: imartinezcuevas
date: 2025-11-27 11:39:00 +0800
categories: [Reddit sentiment analysis]
tags: [FastAPI, PostgreSQL, ]
---

This project implements a scalable microservices platform for ingesting, analyzing, and correcting Reddit posts to generate sentiment-labeled datasets. The platform uses FastAPI, RQ, Redis, PostgreSQL, Docker, Airflow, and Prometheus, providing a robus, observable, and asynchronous system. This article describes the work done, encountered challenges and technicals solutions.

![Diagram](/assets/post_imgs/building-reddit-sentiment-analyzer/high-level-diagram.png){: width="500" height="600" }
_High-level architecture diagram_

## Platform architecture
The system consists of several independent services, orchestrated to handle ingestion, sentiment analysis, correction and dataset generation.
1. API Gateway (FastAPI)
    * Exposes endpoints for keyword search, sentiment retrieval, and manual correction.
    * Implements Redis caching for results and job deduplication.
    * Provides `/metrics` and `/health` endpoints for observability.

    ![Diagram](/assets/post_imgs/building-reddit-sentiment-analyzer/api-gateway-diagram.png){: width="500" height="600" }
    _Diagram zoomed into API Gateway showing endpoints, caching layer, and Redis locks_

2. Reddit Ingestor (asyncpraw + FastAPI worker)
    * Fetches posts asynchronously, normalizing content for downstream processing.
    * Handles rate limtis and posts inconsistencies.
    * Filters posts with missing or malformed fields.

3. Sentiment Analyzer (FastAPI + HuggingFace Transformers)
    * Processes batches of text and returns normalized sentiment labels.
    * Normalizes output labels for consistency.

4. Background Workers (RQ + Redis)
    * Executes ingestion and sentiment analysys jobs asynchronously.
    * Ensures API Gateway remains lightweight and responsive.
    * Implements Redis-based job locking to prevent duplicates.

5. Database (PostgreSQL)
    * Stores manual sentiment corrections and logs dataset generation.

6. AIrflow DAGs
    * Automates dataset generation when a threshold of manual corrections is reached.

## Challenges and technical solutions
During the development of the pipeline, I encountered several technical challenges that required careful resolution. One of the first issues involved job deduplication and cache incosistencies. When multiple API request arrived simultaneously for the same keyword, the system would queue redundant background jobs. This resulted in multiple workers performing identical sentiment analyses, overwriting cached results in Redis. The root cause was that the FastAPI endpoints triggered RQ jobs asynchronously without any per-key locking, and Redis cacing used a naive `Set` operation that didn't account for jobs in progress. The solution involved implementing a Redis-based distributed lock for each keyword. Before queueing a job: The API attempts to acquire a lock. If the lock already exists, The API returns a "queued" status. If not, the API enqueue the job. This ensures that only one worker processes a keyword at a time.

Airflow introduced significant orchestration challenges, particularly related to containerized task execution adn dataset generation. The DAG responsible for generating datasets didn't execute in a persistent process, instead, each task spin-up created a new Airflow worker container. While this ensures isolation, it complicated access to intermediate files. Initially, I attempted to output generated files directly to a host folder, but I couldn't. After experimentation ,the solution involved writing output data to a Docker volume. Once the DAG completes, a separate bash script can copy files from the volume to the desired host directory.

Another issue with Airflow is the dataset size threshold logic. The pipeline is intended to generate a new dataset each time there are at least 50 new sentiment corrections in PostgreSQL. In practice, the DAG can't wait dynamically for this condition without continous pooling. At the moment, the solution is semi-manual: the DAG is triggered by an operator only after verifying the minimum number of corrections.

## Conclusions

Building the postmood sentiment analysis pipeline has been a challenge. One of the central lessons is thtat ephemeral container, while ideal for isolation and reproducibility, introduce non-obvious challengesw in data persistence and workflow automation. I learned that volumes are esential for bridging containers with host storage, but manual interventions and auxiliary scripts are often required unless an event-driven architecture is implemented.

Overall, the project has been a mix of trial, error and iteration. More than just the technical skills, I now better understand how all the pieces fit together and why anticipating edge cases early is critial in real-world systems.