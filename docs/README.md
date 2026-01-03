# ADK Agent Deployment Kit

**Author:** Adri  
**Date:** 2026-01-03  
**Purpose:** Definitive solution for deploying ADK Agents to Vertex AI Agent Engine

---

## 📦 Included Files

| File | Purpose | When to Use |
|------|---------|-------------|
| `ADK_DEPLOYMENT_GUIDE.md` | Complete guide with detailed explanations | First time or troubleshooting |
| `ADK_DEPLOYMENT_CHEATSHEET.md` | Quick command reference | Quick lookup |
| `deploy_agent.py` | Ready-to-use deployment template | Every deployment |
| `test_agent.py` | Ready-to-use testing template | Post-deployment testing |

---

## 🚀 Quick Start (3 Steps)

### 1. Prepare Your Project

```bash
your_project/
├── your_agent_folder/
│   ├── __init__.py      # from .agent import root_agent
│   ├── agent.py         # root_agent = Agent(...)
│   └── .env             # Configuration
├── deploy_agent.py      # ← Copy deploy_agent.py here
└── test_agent.py        # ← Copy test_agent.py here
```

### 2. Deploy

```python
# Edit deploy_agent.py:
PROJECT_ID = "your-project-id"
AGENT_FOLDER = "your_agent_folder"
# ... other variables

# Execute:
python deploy_agent.py
```

### 3. Test

```python
# Edit test_agent.py:
PROJECT_ID = "your-project-id"
TEST_MESSAGES = ["Hello!", "What can you do?"]

# Execute:
python test_agent.py
```

---

## 📝 Detailed Instructions

### Step 1: Initial Setup

1. **Create your agent** following ADK's standard structure:
   ```python
   # your_agent_folder/agent.py
   from google.adk.agents import Agent
   
   root_agent = Agent(
       name="my_agent",
       model="gemini-2.0-flash",
       instruction="You are a helpful agent",
       tools=[...]
   )
   ```

2. **Create `__init__.py`**:
   ```python
   # your_agent_folder/__init__.py
   from .agent import root_agent
   __all__ = ["root_agent"]
   ```

3. **Configure `.env`** (optional but recommended):
   ```bash
   # your_agent_folder/.env
   GOOGLE_GENAI_USE_VERTEXAI="True"
   GOOGLE_CLOUD_PROJECT="your-project-id"
   GOOGLE_CLOUD_LOCATION="us-central1"
   ```

### Step 2: Deployment

1. **Copy the template**:
   ```bash
   cp deploy_agent.py your_project/deploy_agent.py
   ```

2. **Update configuration** in `deploy_agent.py`:
   ```python
   PROJECT_ID = "your-project-id"
   LOCATION = "us-central1"
   STAGING_BUCKET = "gs://your-bucket"
   AGENT_FOLDER = "your_agent_folder"
   AGENT_DISPLAY_NAME = "My Awesome Agent"
   ```

3. **Execute deployment**:
   ```bash
   python deploy_agent.py
   ```

4. **Verify** that `agent_resource_name.txt` was created with the resource ID

### Step 3: Testing

1. **Copy the template**:
   ```bash
   cp test_agent.py your_project/test_agent.py
   ```

2. **Update configuration** (if needed):
   ```python
   PROJECT_ID = "your-project-id"
   LOCATION = "us-central1"
   # AGENT_RESOURCE_NAME is read automatically from agent_resource_name.txt
   
   TEST_MESSAGES = [
       "Hello!",
       "What can you help me with?",
       "Tell me a joke",
   ]
   ```

3. **Run tests**:
   ```bash
   python test_agent.py
   ```

---

## 🔧 Troubleshooting

### Error during deployment

1. **Check complete guide**: `ADK_DEPLOYMENT_GUIDE.md`
2. **Verify logs**:
   ```bash
   gcloud logging read "resource.type=aiplatform.googleapis.com/ReasoningEngine" \
     --project=PROJECT_ID --limit=50
   ```

### Error during testing

1. **Verify agent was deployed**:
   ```bash
   gcloud ai reasoning-engines list --region=us-central1
   ```

2. **Test in UI**:
   - Go to https://console.cloud.google.com/vertex-ai/agents/
   - Find your agent
   - Use the chat interface

### Common Errors

See the **Troubleshooting** section in `ADK_DEPLOYMENT_GUIDE.md` for:
- `ModuleNotFoundError`
- `'AgentEngine' object has no attribute 'query'`
- Permission issues
- And more...

---

## 💡 Tips and Best Practices

### 1. Consistent file structure

Always use the same structure:
```
project/
├── agent_name/
│   ├── __init__.py
│   ├── agent.py
│   └── .env
├── deploy_agent.py
├── test_agent.py
└── agent_resource_name.txt  (auto-generated)
```

### 2. Versioning

Include version in display name:
```python
AGENT_DISPLAY_NAME = "My Agent v1.2.0"
```

### 3. Test locally first

Before deploying, test locally:
```python
from your_agent_folder.agent import root_agent
response = root_agent.send_message("test")
print(response)
```

### 4. Secrets management

DON'T commit `.env` to git:
```bash
# .gitignore
.env
agent_resource_name.txt
```

### 5. CI/CD

Templates are CI/CD compatible:
```yaml
# .github/workflows/deploy.yml
- name: Deploy Agent
  run: |
    python deploy_agent.py
  env:
    PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
```

---

## 📚 Additional Resources

### Documentation
- [ADK Docs](https://google.github.io/adk-docs/)
- [Agent Engine Docs](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine)

### Useful Commands

```bash
# Authentication
gcloud auth application-default login

# List deployed agents
gcloud ai reasoning-engines list --region=us-central1

# View agent details
gcloud ai reasoning-engines describe RESOURCE_ID --region=us-central1

# Delete an agent
gcloud ai reasoning-engines delete RESOURCE_ID --region=us-central1

# View logs
gcloud logging read "resource.type=aiplatform.googleapis.com/ReasoningEngine" \
  --project=PROJECT_ID --limit=50 --format=json
```

---

## 🎯 Complete Example Workflow

```bash
# 1. Create new project
mkdir my_new_agent
cd my_new_agent

# 2. Create structure
mkdir weather_agent
touch weather_agent/__init__.py
touch weather_agent/agent.py

# 3. Copy templates
cp ../deploy_agent.py deploy_agent.py
cp ../test_agent.py test_agent.py

# 4. Implement the agent
# ... edit weather_agent/agent.py ...

# 5. Configure deployment
# ... edit deploy_agent.py ...

# 6. Deploy
python deploy_agent.py

# 7. Test
python test_agent.py

# 8. Iterate
# ... make changes to agent ...
# ... deploy again ...
```

---

## ⚠️ IMPORTANT

### DON'T use the ADK CLI:
```bash
❌ adk deploy agent_engine ...  # Has bugs
```

### DO use programmatic deployment:
```python
✅ python deploy_agent.py  # Uses API directly
```

---

## 📞 Support

If you encounter problems:
1. Review `ADK_DEPLOYMENT_GUIDE.md`
2. Check `ADK_DEPLOYMENT_CHEATSHEET.md`
3. Review logs in Google Cloud Console
4. Check [official documentation](https://google.github.io/adk-docs/)

---

**Last updated:** 2026-01-03  
**Maintainer:** Adri - Applied AI Engineer @ Aselvia
