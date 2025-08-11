---
title: "Setting up a clean, scalable Python project"
description: "How to set up a python project"
author: imartinezcuevas
date: 2025-08-11 12:44:00 +0800
categories: [Backend]
tags: [Backend]
---

## Introduction
I've work on several projects using the traditional `venv + pip` setup, and let's be honest, it gets tedious fast. Every time you add or upgrade a dependency, you have  to remember to regenerate `requirements.txt` and make sure everyone on the team installs it correctly.

For this project, I want to do things differently. I want a clean setup that doesn't just work today but will be easy to maintain and extend months. And sice I also want to learn new things, I'm switching to Poetry for project management.

Poetry is a modern Python packaging and dependency management tool that replace the old `pip` + `venv` workflow. It handles virtual environments automatically, locks dependencies for reproducibility, and manages everythng from a single file.

In this article I want to show how I will manage my packages or this project, I'll also adress common pitfalls like:
* Dependency conflicts and "version drift".
* Inconsistent coding styles across the team.
* weak type safety leading to runtime errors.
* Manual, error-prone deplyment setups.

## Setting up the project with Poetry
After deciding on Poetry, the first thing I have to do is create a dedicated folder for the project:

```console
mkdir jobmatcher
cd jobmatcher
poetry init
```

The `poetry init` command walks you through creating a `pyproject.toml` file, this replaces the traditional `requirements.txt` and `setup.py` combination. This file will hold all my dependencies, the Python version, and any tool configuration I need later. 

Then I have to create the folder structure I decided the last article:
```console
mkdir -p src/{core/{jobs,resumes,scoring,users,recommendations},services,infrastructure,api,worker,config,shared}
touch src/__init__.py
touch src/core/__init__.py
touch src/services/__init__.py
touch src/infrastructure/__init__.py
touch src/api/__init__.py
touch src/worker/__init__.py
touch src/config/__init__.py
touch src/shared/__init__.py
```

Now I can start adding the main libraries I'm going to use. I'm choosing FastAPI as my framework and MongoDB as my database, with Beanie as the ODM since it works great with Pydantic models and async code.

```console
poetry add fastapi uvicorn[standard] beanie motor pydantic[dotenv] pydantic-settings httpx
```

Here's why I'm installing each one:
* `fastapi` --> The main framework for my API.
* `uvicorn` --> The server to run FastAPI.
* `beanie` --> ODM for MongoDB, async-friendly.
* `motor` --> Async driver for MongoDB (Beanie uses it).
* `pydantic` --> Data validation and `.env` configuration.
* `httpx` --> Async HTTP client for web scraping and API calls.

And before I even write the first endpoint, I want to make sure the code will be consistent and easy to maintain. So I'm adding the dev tools right away:

```console
poetry add --dev black ruff mypy pytest pytest-asyncio pre-commit
```
* `black` --> Auto-formatting.
* `ruff` --> Linting and import sorting.
* `mypy` --> Static type checks to catch mistakes early.
* `pytest` --> Testing framework.
* `pytest-asyncio` --> Async test support.
* `pre-commit` --> Automatically runs checks before commits.

## Configuring `pyproject.toml` and code quality tools
Not that I've installed all the dependencies, I need to make sure my tools know how to behave. The nice thing about Poetry is that most Python tools can be configured directly inside its file.

With `pyproject.toml` we add these sections:
```toml
[tool.black]
line-length = 88
target-version = ['py312']

[tool.ruff]
line-length = 88
select = ["E", "F", "B", "I", "UP"]
ignore = ["E501"]

[tool.mypy]
python_version = 3.12
strict = true

[tool.pytest.ini_options]
testpaths = ["src"]

[tool.poetry.scripts]
runserver = "api.main:run"
```

Lastly, I want to make sure that whenever I commit code, it's already formatted, linted and type-checked. For that, I set up pre-commit. Created a file called `.pre-commit-config.yaml` in the root folder and add:

```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 25.1.0
    hooks:
      - id: black
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.12.8
    hooks:
      - id: ruff
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.17.1
    hooks:
      - id: mypy
```

Then we install the hooks:
```console
pre-commit install
```

From now on, every t ime I try to commit, these tools will run automatically. If something fails, the commit is blocked until I fix it. That means no more unformatted or type-broken code making it into the repo.

## Setting up MongoDB with Beanie and Encironment variables
I don't want to hardcode my MongoDB URI anywhere in the code, so I create a `.env` file at the root of the project:
```console
MONGODB_URI=mongodb://example:27017/jobmatcherdb
```

To keep environment config clean, I create a new file: `src/config/settings.py` and inside I write:
```python
from pydantic import BaseSettings

class Settings(BaseSettings):
    mongodb_uri: str

    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

settings = Settings()
```

This will automatically load `.env` and give access to the URI. We also have to setup the mongoDB connection and initialize Beanie. I'll do all of this in the file `db.py`:

```python
from motor.motor_asyncio import AsyncIOMotorClient
from beanie import init_beanie
from config.settings import settings
import asyncio

async def init_db():
    client = AsyncIOMotorClient(settings.mongodb_uri)
    db = client.get_default_database()
    await init_beanie(database=db)
```

Lastly, we have to connect to the database on FastAPI startup. We achive this creating an even on startup:
```python
from fastapi import FastAPI
from infrastructure.db import init_db

app = FastAPI()

@app.on_event("startup")
async def on_startup():
    await init_db()

@app.get("/")
async def root():
    return {"message": "Backend is running and DB connected!"}
```

## Conclusion
With the project structure in place, Poetry managing dependencies, pre-commit hooks configured for quality, and MongoDB connected cleanly via Beanie and environment variables, we have a solid, scalable foundation ready to start real development.

## References 
https://dev.to/leapcell/poetry-pip-venv-heres-why-developers-are-switching-5005
https://medium.com/@deepakcoder80/why-i-choose-poetry-over-venv-for-managing-python-dependencies-95cd7eb50223
https://medium.com/@pdx.lucasm/python-poetry-f8cbab0eef94
https://dev.to/techishdeep/maximize-your-python-efficiency-with-pre-commit-a-complete-but-concise-guide-39a5
https://medium.com/internet-of-technology/beautify-your-python-code-with-pre-commit-linters-a-step-by-step-guide-d63604d6120b