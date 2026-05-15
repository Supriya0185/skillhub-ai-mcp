Project name:

skillhub-ai-mcp

Folder structure:

skillhub-ai-mcp/
│
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── skills.py
│   ├── schemas.py
│   └── config.py
│
├── requirements.txt
├── Procfile
├── manifest.yml
├── README.md
└── .gitignore
1. README.md
# SkillHub AI - MCP Based Enterprise Skill Marketplace

SkillHub AI is a simple learning project to understand how FastAPI, AI Skills, MCP, and SAP Cloud Foundry deployment work together.

## Project Goal

The goal of this project is to build a small platform where reusable AI skills can be listed, executed, and later exposed through MCP.

## What is a Skill?

A skill is a reusable task.

Examples:

- Email Summary Skill
- Log Diagnosis Skill
- SQL Explain Skill
- Resume Rewrite Skill

## What is MCP?

MCP stands for Model Context Protocol.

In simple words, MCP helps an AI assistant connect to external tools and skills in a standard way.

## First Version Scope

In the first version, this project will have:

- FastAPI backend
- Two simple skills
- Skill listing API
- Skill execution API
- Health check API
- Production-style folder structure
- SAP Cloud Foundry deployment files

## Initial Skills

### 1. Email Summary Skill

Takes email or project text and returns a simple summary and action items.

### 2. Log Diagnosis Skill

Takes application error logs and returns root cause and fix steps.

## Architecture

```text
User / Browser / Swagger UI
        ↓
FastAPI Backend
        ↓
Skill Registry
        ↓
Selected Skill Function
        ↓
JSON Response
Future MCP Flow
AI Assistant
        ↓
MCP Server
        ↓
SkillHub API
        ↓
Selected Skill
        ↓
Response
Tech Stack
Purpose	Tool
Backend API	FastAPI
Local Server	Uvicorn
Production Server	Gunicorn
Validation	Pydantic
Deployment	SAP Cloud Foundry
Version Control	GitHub
Language	Python
Planned Folder Structure
skillhub-ai-mcp/
│
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── skills.py
│   ├── schemas.py
│   └── config.py
│
├── requirements.txt
├── Procfile
├── manifest.yml
├── README.md
└── .gitignore
Local Setup

Create virtual environment:

python -m venv venv

Activate virtual environment on Windows:

.\venv\Scripts\activate

Install dependencies:

python -m pip install --upgrade pip
python -m pip install -r requirements.txt

Run locally:

uvicorn app.main:app --reload
Local URL
http://127.0.0.1:8000
Swagger UI
http://127.0.0.1:8000/docs
Main APIs
API	Method	Purpose
/	GET	Home route
/health	GET	Health check
/skills	GET	List all skills
/skills/{skill_id}/execute	POST	Execute selected skill
Example Request

Skill ID:

log-diagnosis

Request body:

{
  "input_text": "App crashed with memory quota exceeded and exit status 137"
}
Example Response
{
  "status": "success",
  "skill_id": "log-diagnosis",
  "skill_name": "Log Diagnosis Skill",
  "output": {
    "root_cause": "Application memory limit exceeded.",
    "fix_steps": [
      "Check recent logs using cf logs app-name --recent",
      "Increase memory using cf scale app-name -m 1G",
      "Restart the application using cf restart app-name"
    ],
    "prevention_tip": "Set proper memory in manifest.yml before deployment."
  }
}
SAP Cloud Foundry Deployment

Login to SAP Cloud Foundry:

cf login

Push app:

cf push

Check app status:

cf apps

View logs:

cf logs skillhub-ai-mcp --recent
Project Status
Step 1: Repository setup
Step 2: Local folder structure
Step 3: FastAPI app
Step 4: Skill execution API
Step 5: Production files
Step 6: SAP Cloud Foundry deployment
Step 7: MCP integration

---

# 2. `.gitignore`

```gitignore
# Python virtual environment
venv/
.env/

# Python cache
__pycache__/
*.py[cod]
*$py.class

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Environment variables
.env
.env.local

# Logs
*.log

# VS Code
.vscode/

# PyCharm
.idea/

# OS files
.DS_Store
Thumbs.db

# SQLite database files
*.db
*.sqlite3
3. requirements.txt
fastapi==0.115.6
uvicorn==0.34.0
gunicorn==23.0.0
pydantic==2.10.4
4. Procfile
web: gunicorn app.main:app -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:$PORT
5. manifest.yml
applications:
  - name: skillhub-ai-mcp
    memory: 512M
    instances: 1
    buildpacks:
      - python_buildpack
    command: gunicorn app.main:app -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:$PORT
    env:
      APP_NAME: SkillHub AI MCP
      APP_ENV: cloud
      APP_VERSION: 1.0.0
6. app/__init__.py
"""
SkillHub AI application package.

This file makes the app folder a Python package.
"""
7. app/config.py
import os


class Settings:
    """
    Application configuration.

    Values are read from environment variables.
    If environment variables are not available, default values are used.
    """

    APP_NAME: str = os.getenv("APP_NAME", "SkillHub AI MCP")
    APP_ENV: str = os.getenv("APP_ENV", "local")
    APP_VERSION: str = os.getenv("APP_VERSION", "1.0.0")


settings = Settings()
8. app/schemas.py
from pydantic import BaseModel, Field
from typing import Any, Dict, List, Optional


class SkillRequest(BaseModel):
    """
    Request model for executing a skill.
    """

    input_text: str = Field(
        ...,
        min_length=3,
        description="Input text that will be processed by the selected skill"
    )


class SkillInfo(BaseModel):
    """
    Response model for skill information.
    """

    skill_id: str
    name: str
    description: str


class SkillListResponse(BaseModel):
    """
    Response model for listing skills.
    """

    total_skills: int
    skills: List[SkillInfo]


class SkillExecutionResponse(BaseModel):
    """
    Response model for skill execution.
    """

    status: str
    skill_id: str
    skill_name: str
    input: str
    output: Dict[str, Any]
    latency_ms: float


class HealthResponse(BaseModel):
    """
    Response model for health check.
    """

    status: str
    app: str
    environment: str
    version: str


class ErrorResponse(BaseModel):
    """
    Response model for errors.
    """

    status: str
    message: str
    details: Optional[str] = None
9. app/skills.py
from typing import Dict, Any, Callable


def email_summary_skill(input_text: str) -> Dict[str, Any]:
    """
    Email Summary Skill.

    This is a simple rule-based skill.
    It does not use any paid LLM or API.

    Input:
        Email or project communication text

    Output:
        Summary, action items, priority, and detected deadline
    """

    lower_text = input_text.lower()

    action_items = []

    if "test" in lower_text or "testing" in lower_text:
        action_items.append("Testing activity found")

    if "deploy" in lower_text or "deployment" in lower_text:
        action_items.append("Deployment activity found")

    if "validate" in lower_text or "validation" in lower_text:
        action_items.append("Validation activity found")

    if "review" in lower_text:
        action_items.append("Review activity found")

    if "friday" in lower_text:
        deadline = "Friday"
    elif "monday" in lower_text:
        deadline = "Monday"
    elif "today" in lower_text:
        deadline = "Today"
    elif "tomorrow" in lower_text:
        deadline = "Tomorrow"
    else:
        deadline = "Not detected"

    if "urgent" in lower_text or "immediately" in lower_text or "asap" in lower_text:
        priority = "High"
    elif "by friday" in lower_text or "by tomorrow" in lower_text:
        priority = "Medium"
    else:
        priority = "Low"

    return {
        "summary": "This text contains project communication and possible work-related action items.",
        "action_items": action_items if action_items else ["No clear action item detected"],
        "priority": priority,
        "deadline": deadline
    }


def log_diagnosis_skill(input_text: str) -> Dict[str, Any]:
    """
    Log Diagnosis Skill.

    This is a simple rule-based skill for common deployment/runtime errors.

    Input:
        Application error log text

    Output:
        Root cause, fix steps, and prevention tip
    """

    lower_text = input_text.lower()

    if "memory quota exceeded" in lower_text or "exit status 137" in lower_text or "memory" in lower_text:
        return {
            "root_cause": "Application memory limit exceeded.",
            "fix_steps": [
                "Check recent logs using: cf logs app-name --recent",
                "Increase memory using: cf scale app-name -m 1G",
                "Restart the application using: cf restart app-name"
            ],
            "prevention_tip": "Set proper memory in manifest.yml before deployment."
        }

    if "port" in lower_text or "failed to bind" in lower_text:
        return {
            "root_cause": "Application may not be listening on the correct cloud platform port.",
            "fix_steps": [
                "Use the PORT environment variable provided by the platform",
                "Start the app with host 0.0.0.0",
                "For FastAPI, use: uvicorn app.main:app --host 0.0.0.0 --port $PORT",
                "Restart the app after fixing the start command"
            ],
            "prevention_tip": "Cloud Foundry apps must listen on the port assigned by the platform."
        }

    if "module not found" in lower_text or "modulenotfounderror" in lower_text:
        return {
            "root_cause": "A required Python dependency is missing.",
            "fix_steps": [
                "Check the missing module name in the logs",
                "Add the required package to requirements.txt",
                "Run local test using: python -m pip install -r requirements.txt",
                "Push the app again using: cf push"
            ],
            "prevention_tip": "Keep requirements.txt updated with all required packages."
        }

    if "crash" in lower_text or "crashed" in lower_text:
        return {
            "root_cause": "Application crashed during startup or runtime.",
            "fix_steps": [
                "Run: cf logs app-name --recent",
                "Check startup command in Procfile or manifest.yml",
                "Check memory allocation",
                "Check dependency installation",
                "Restart the app after fixing the issue"
            ],
            "prevention_tip": "Always test the app locally before deploying to cloud."
        }

    return {
        "root_cause": "Unknown error pattern.",
        "fix_steps": [
            "Run: cf logs app-name --recent",
            "Check application startup command",
            "Check memory allocation",
            "Check port configuration",
            "Check requirements.txt dependencies"
        ],
        "prevention_tip": "Add more known error rules to improve diagnosis accuracy."
    }


SkillFunction = Callable[[str], Dict[str, Any]]


SKILLS: Dict[str, Dict[str, Any]] = {
    "email-summary": {
        "name": "Email Summary Skill",
        "description": "Summarizes email or project communication text and extracts basic action items.",
        "function": email_summary_skill
    },
    "log-diagnosis": {
        "name": "Log Diagnosis Skill",
        "description": "Diagnoses common deployment and runtime errors from application logs.",
        "function": log_diagnosis_skill
    }
}
10. app/main.py
import logging
import time
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware

from app.config import settings
from app.schemas import SkillRequest
from app.skills import SKILLS


logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(name)s | %(message)s"
)

logger = logging.getLogger("skillhub-ai-mcp")


app = FastAPI(
    title=settings.APP_NAME,
    version=settings.APP_VERSION,
    description="SkillHub AI - Production-style mini app for learning FastAPI, AI Skills, MCP, and SAP Cloud Foundry"
)


app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/")
def root():
    """
    Home endpoint.

    Used to verify that the application is running.
    """

    logger.info("Root endpoint called")

    return {
        "message": "SkillHub AI MCP app is running successfully",
        "app": settings.APP_NAME,
        "environment": settings.APP_ENV,
        "version": settings.APP_VERSION,
        "docs": "/docs",
        "health": "/health",
        "skills": "/skills"
    }


@app.get("/health")
def health_check():
    """
    Health check endpoint.

    In production, this type of endpoint is used by platforms
    to check whether the application is alive.
    """

    logger.info("Health check endpoint called")

    return {
        "status": "healthy",
        "app": settings.APP_NAME,
        "environment": settings.APP_ENV,
        "version": settings.APP_VERSION
    }


@app.get("/skills")
def list_skills():
    """
    List all available skills.

    This endpoint shows the skill catalog.
    """

    logger.info("List skills endpoint called")

    skill_list = []

    for skill_id, skill_data in SKILLS.items():
        skill_list.append({
            "skill_id": skill_id,
            "name": skill_data["name"],
            "description": skill_data["description"]
        })

    return {
        "total_skills": len(skill_list),
        "skills": skill_list
    }


@app.post("/skills/{skill_id}/execute")
def execute_skill(skill_id: str, request: SkillRequest):
    """
    Execute selected skill.

    User provides a skill_id and input_text.
    The backend finds the correct skill and runs it.
    """

    start_time = time.time()

    logger.info(f"Execution request received for skill_id={skill_id}")

    if skill_id not in SKILLS:
        logger.warning(f"Skill not found: {skill_id}")
        raise HTTPException(
            status_code=404,
            detail=f"Skill '{skill_id}' not found"
        )

    selected_skill = SKILLS[skill_id]
    skill_function = selected_skill["function"]

    try:
        result = skill_function(request.input_text)

        latency_ms = round((time.time() - start_time) * 1000, 2)

        logger.info(
            f"Skill executed successfully | skill_id={skill_id} | latency_ms={latency_ms}"
        )

        return {
            "status": "success",
            "skill_id": skill_id,
            "skill_name": selected_skill["name"],
            "input": request.input_text,
            "output": result,
            "latency_ms": latency_ms
        }

    except Exception as error:
        logger.error(f"Skill execution failed | skill_id={skill_id} | error={str(error)}")

        raise HTTPException(
            status_code=500,
            detail="Skill execution failed"
        )


@app.get("/version")
def get_version():
    """
    Version endpoint.

    Useful for confirming which version is running in local/cloud.
    """

    return {
        "app": settings.APP_NAME,
        "version": settings.APP_VERSION,
        "environment": settings.APP_ENV
    }
Commands to run locally

Open PowerShell inside project folder:

python -m venv venv

Activate:

.\venv\Scripts\activate

Install:

python -m pip install --upgrade pip
python -m pip install -r requirements.txt

Run:

uvicorn app.main:app --reload

Open:

http://127.0.0.1:8000

Swagger:

http://127.0.0.1:8000/docs
Test input 1: Email Summary Skill

Endpoint:

POST /skills/email-summary/execute

Body:

{
  "input_text": "Hi team, please complete testing by Friday. We also need to validate deployment before release."
}
Test input 2: Log Diagnosis Skill

Endpoint:

POST /skills/log-diagnosis/execute

Body:

{
  "input_text": "App crashed with memory quota exceeded and exit status 137"
}

For now add these files exactly. After this, we will run it locally and test in Swagger.