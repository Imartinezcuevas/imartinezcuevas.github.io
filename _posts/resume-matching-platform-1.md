---
title: "Building a Scalable Backend in Python: A Modular Monolith Approach"
description: "How I approached building a scalable backend for a job matching platform"
author: imartinezcuevas
date: 2025-07-28 12:32:00 +0800
categories: [AI, Backend]
tags: [Backend, FastAPI, AI]
---

## Introduction
Searching for a job today feels like competing in a race where hundreds of applicants flood every new listing within hours. If your resume isn't one of the first seen by a recruiter, your chances drop dramatically.

Having faced this challenge firsthand, I decided to build an AI-powered resume matching platform. It will automatically scrape job listings, parse resumes, score compatibility and recommend relevant jobs. But since I had never built something like this before, many questions came up: Where do I start? How do I design a system with these capabilities? How do I keep it scalable and maintainable over time?

This article walks through my thought process on tackling these challenges with a Modular Monolith architecture. I chose this approach because it balances the simplicity and productivity of a monolith with the modularity and maintainability of microservices. More importantly, it allows me to focus on learning about FastAPI, web scraping, task scheduling, etc., rather than getting lost in infrastructure complexity.

## System requirements
Before jumping into code or frameworks, I asked myself: what are the essential capabilities this platform must have?
 * **High-volume, multi-source data ingestion:** The system needs to scrape job listing regularly from multiple websites and keep the data fresh.
 * **Data normalization and deduplication:** Every job board formats listing differently. The backend must standardize this data and detect duplicates.
 * **Matching and scoring:** I want the system to be able to quantify how well a user's resume fits a job, highlight skill gaps, and suggest skills worth learning to improve chances.
 * **Recommendations and notifications:** The system should proactively suggest jobs and send timely alerts.

We know now, developing this would involve multiple workflows, from scraping and parsing to scoring and notifying users. I want to be able to manage all this complexity without losing agility or creating a maintenance nightmare.

## Modular Monolith over Microservices
Not knowing where to start when building an application, the first thing that came to mind was: what architecture should I use? Since microservices are all over the place lately, it was my first choice. But after doing some research, I found many articles recommending not to start with microservices, as they add a lot of unnecessary complexity early on. Instead, it's better to begin with a modular monolith, and if the need arises in the future, gradually transition to microservices.

Some of the articles I read:
* [Modular Monolith: A Primer](https://www.kamilgrzybek.com/blog/posts/modular-monolith-primer)
* [Modular Monolithic vs. Microservices](https://www.fullstack.com/labs/resources/blog/modular-monolithic-vs-microservices)
* [Modular Monolith: Architectural Drivers](https://www.kamilgrzybek.com/blog/posts/modular-monolith-architectural-drivers)
* [What Is a Modular Monolith?](https://www.milanjovanovic.tech/blog/what-is-a-modular-monolith)

Here's why it won for me:
* **Complexity**: A monolith keeps everything in one place, easier to understand and debug.
* **Deployability**: One deployment. Rolling back or pushing is straightforward. 
* **Performance**: No network hops between components. Everything runs in-process, improving speed and reducing latency.
* **Team size**: Microservices shine when you have multiple teams owning different services. Right now, just myself benefits more from a modular monolith.

## High-level project structure
This is how I organized the code to keep things manageable:
```console
/src/
  ├── core/               # Domain logic
  │   ├── jobs/
  │   ├── resumes/
  │   ├── scoring/
  │   ├── users/
  │   └── recommendations/
  ├── services/           # Application layer (use cases / orchestration)
  ├── infrastructure/     # External systems (scrapers, DB, email, parsers)
  ├── api/                # FastAPI routes and controllers
  ├── worker/             # Background jobs (e.g., scraping, scoring)
  ├── config/             # Dependency injection, settings, logging
  └── shared/             # Utilities (token management, hashing, etc.)
```
Without clear boundaries and discipline, the backend can quickly become unmanageable. To prevent this, I used architectural principles from DDD, Hexagonal and Onion architecture.
* DDD to define clear business domains (/core/).
* Hexagonal to isolate core business logic from external systems like databases, scrapers, etc. (/infrastructure/)

At the end of the day, architecture is just a tool, not the goal. The real objective is to solve a problem, not get caught up in discussion about frameworks, design patterns or whether microservices are "better" than monoliths.
