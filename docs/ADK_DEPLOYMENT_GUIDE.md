# Complete Guide: Deploying ADK Agents to Vertex AI Agent Engine

**Author:** Adri  
**Date:** 2026-01-03  
**Version:** 1.0

---

## 📋 Table of Contents

1. [The Common Problem](#the-common-problem)
2. [The Solution](#the-solution)
3. [Ready-to-Use Scripts](#ready-to-use-scripts)
4. [Testing Deployed Agents](#testing-deployed-agents)
5. [Troubleshooting](#troubleshooting)
6. [Pre-Deployment Checklist](#pre-deployment-checklist)

---

## ❌ The Common Problem

### Error: `ModuleNotFoundError: No module named 'agent_engine_app'`

When using the ADK CLI:
```bash
adk deploy agent_engine --project=PROJECT_ID --region=REGION --staging_bucket=gs://BUCKET agent_folder
```

**Deployment fails with:**
```
ModuleNotFoundError: No module named 'agent_engine_app'
```

### Why Does It Fail?

The ADK CLI has a bug:
1. Creates temporary files (`agent_engine_app.py`)
2. Uploads them to GCS
3. **FAILS** to import them on the remote server
4. The `agent_engine_app` module is not found

---

## ✅ The Solution

### Don't use the CLI, use programmatic deployment

Instead of `adk deploy agent_engine`, create a Python script that uses the API directly.

**Advantages:**
- ✅ Bypasses CLI bugs
- ✅ Full control over requirements and packages
- ✅ Avoids `absolufy-imports` issues
- ✅ More reliable and reproducible

---

## 📝 Ready-to-Use Scripts

### Script 1: Deployment (deploy_agent.py)

```python
"""
Deployment script for ADK Agents
Uses Python API directly to avoid CLI bugs
"""
import vertexai
from vertexai.agent_engines import AdkApp

# ============================================================================
# CONFIGURATION - UPDATE THESE VALUES
# ============================================================================
PROJECT_ID = "your-project-id"
LOCATION = "us-central1"
STAGING_BUCKET = "gs://your-bucket"
AGENT_DISPLAY_NAME = "My Agent"
AGENT_DESCRIPTION = "Description of what the agent does"

# ============================================================================
# DEPLOYMENT
# ============================================================================

# Initialize Vertex AI
vertexai.init(
    project=PROJECT_ID,
    location=LOCATION,
    staging_bucket=STAGING_BUCKET
)

# Import your agent (adjust the import according to your structure)
from your_agent_folder.agent import root_agent

print("🚀 Starting deployment...")
print(f"📦 Agent: {root_agent.name}")
print(f"🔧 Model: {root_agent.model}")

# Create AdkApp wrapper
app = AdkApp(agent=root_agent)

# Deploy to Agent Engine
try:
    from vertexai import agent_engines
    
    remote_app = agent_engines.create(
        app,
        requirements=[
            "google-cloud-aiplatform[adk,agent_engines]>=1.132.0",
        ],
        # ⭐ CRITICAL: Include your agent folder as extra package
        extra_packages=["./your_agent_folder"],
        display_name=AGENT_DISPLAY_NAME,
        description=AGENT_DESCRIPTION,
    )
    
    print("\n" + "="*70)
    print("✅ DEPLOYMENT SUCCESSFUL!")
    print("="*70)
    print(f"\n📍 Resource name:")
    print(f"   {remote_app.resource_name}")
    
    resource_id = remote_app.resource_name.split('/')[-1]
    console_url = f"https://console.cloud.google.com/vertex-ai/agents/locations/{LOCATION}/agent-engines/{resource_id}?project={PROJECT_ID}"
    
    print(f"\n🔗 View in console:")
    print(f"   {console_url}")
    
    print(f"\n🔧 To use in another script:")
    print(f"   from vertexai import agent_engines")
    print(f"   agent = agent_engines.get('{remote_app.resource_name}')")
    
    print("\n" + "="*70)
    
except Exception as e:
    print(f"\n❌ Deployment failed: {e}")
    import traceback
    traceback.print_exc()
```

### Script 2: Testing (test_agent.py)

```python
"""
Script to test a deployed agent in Agent Engine
Uses correct ASYNC methods
"""
import asyncio
import vertexai

# ============================================================================
# CONFIGURATION - UPDATE THESE VALUES
# ============================================================================
PROJECT_ID = "your-project-id"
LOCATION = "us-central1"
AGENT_RESOURCE_NAME = "projects/.../locations/.../reasoningEngines/..."

# ============================================================================
# TESTING
# ============================================================================

vertexai.init(project=PROJECT_ID, location=LOCATION)

from vertexai import agent_engines

async def test_agent():
    print("🔍 Getting deployed agent...")
    remote_app = agent_engines.get(AGENT_RESOURCE_NAME)
    
    print(f"✅ Agent found: {remote_app.resource_name}")
    print("\n" + "="*70)
    
    # Create a session
    print("\n📝 Creating session...")
    session = await remote_app.async_create_session(user_id="test_user")
    print(f"✅ Session created: {session['id']}")
    
    # Test 1: Send a message
    print("\n🧪 Test: Sending message to agent...")
    print("-" * 70)
    
    async for event in remote_app.async_stream_query(
        user_id="test_user",
        session_id=session["id"],
        message="Your test message here",
    ):
        print(event)
    
    print("\n" + "="*70)
    print("✅ Test completed!")

# Run the test
if __name__ == "__main__":
    asyncio.run(test_agent())
```

### Script 3: Pre-Deployment Verification (verify_setup.py)

```python
"""
Verifies everything is ready before deployment
"""
import os
import sys
from pathlib import Path

print("🔍 Verifying setup for deployment...")
print("=" * 70)

# ============================================================================
# CONFIGURATION - UPDATE THESE VALUES
# ============================================================================
AGENT_FOLDER = "your_agent_folder"

# ============================================================================
# VERIFICATION
# ============================================================================

# 1. File structure
print(f"\n1️⃣ Verifying file structure...")
required_files = [
    f"{AGENT_FOLDER}/__init__.py",
    f"{AGENT_FOLDER}/agent.py",
]

all_exist = True
for file_path in required_files:
    exists = Path(file_path).exists()
    status = "✅" if exists else "❌"
    print(f"   {status} {file_path}")
    if not exists:
        all_exist = False

if not all_exist:
    print("\n❌ Required files are missing!")
    sys.exit(1)

# 2. Verify __init__.py
print(f"\n2️⃣ Verifying {AGENT_FOLDER}/__init__.py...")
init_file = Path(f"{AGENT_FOLDER}/__init__.py")
init_content = init_file.read_text()

if "root_agent" in init_content:
    print("   ✅ __init__.py exports root_agent")
else:
    print("   ❌ __init__.py does NOT export root_agent!")
    print(f"   Add this to {AGENT_FOLDER}/__init__.py:")
    print("""
from .agent import root_agent
__all__ = ["root_agent"]
""")
    sys.exit(1)

# 3. Verify agent can be imported
print(f"\n3️⃣ Verifying agent can be imported...")
try:
    # Adjust the import according to your structure
    exec(f"from {AGENT_FOLDER}.agent import root_agent")
    print(f"   ✅ Agent imported successfully")
except Exception as e:
    print(f"   ❌ Import error: {e}")
    sys.exit(1)

# 4. Verify GCP credentials
print("\n4️⃣ Verifying GCP credentials...")
try:
    import google.auth
    credentials, project = google.auth.default()
    print(f"   ✅ Credentials found for project: {project}")
except Exception as e:
    print(f"   ⚠️  Warning: {e}")
    print("   Make sure you've run: gcloud auth application-default login")

print("\n" + "=" * 70)
print("✅ All verifications passed!")
print("You can run: python deploy_agent.py")
```

---

## 🧪 Testing Deployed Agents

### Available Methods

Deployed agents in Agent Engine have these methods:

```python
# ❌ DO NOT EXIST
remote_app.query()          
remote_app.send_message()   

# ✅ CORRECT METHODS (all ASYNC)
await remote_app.async_create_session(user_id)
await remote_app.async_stream_query(user_id, session_id, message)
await remote_app.async_list_sessions(user_id)
await remote_app.async_get_session(user_id, session_id)
await remote_app.async_delete_session(user_id, session_id)
```

### Complete Usage Example

```python
import asyncio
import vertexai
from vertexai import agent_engines

vertexai.init(project="PROJECT_ID", location="LOCATION")

async def chat_with_agent():
    # 1. Get the deployed agent
    remote_app = agent_engines.get("RESOURCE_NAME")
    
    # 2. Create session
    session = await remote_app.async_create_session(user_id="user_123")
    session_id = session["id"]
    
    # 3. Multi-turn conversation
    messages = [
        "Hello, who are you?",
        "What can you help me with?",
        "Tell me a joke"
    ]
    
    for msg in messages:
        print(f"\n👤 User: {msg}")
        print("🤖 Agent: ", end="")
        
        async for event in remote_app.async_stream_query(
            user_id="user_123",
            session_id=session_id,
            message=msg
        ):
            # Process stream events
            if hasattr(event, 'content'):
                print(event.content, end="")
        
        print()  # New line

asyncio.run(chat_with_agent())
```

### Testing via UI (Easier)

The quickest way to test:

1. Go to: `https://console.cloud.google.com/vertex-ai/agents/`
2. Click on your deployed agent
3. Use the chat interface to test

---

## 🔧 Troubleshooting

### Error: `ModuleNotFoundError: No module named 'agent_engine_app'`

**Cause:** You used the CLI `adk deploy agent_engine`  
**Solution:** Use programmatic deployment (Script 1)

---

### Error: `'AgentEngine' object has no attribute 'query'`

**Cause:** Tried to use `.query()` on a deployed agent  
**Solution:** Use `.async_stream_query()` (see Script 2)

---

### Error: `No module named 'your_agent_folder'`

**Cause:** Didn't include `extra_packages` in deployment  
**Solution:** Add this to `agent_engines.create()`:
```python
extra_packages=["./your_agent_folder"]
```

---

### Error: `cannot import name 'root_agent'`

**Cause:** Your `__init__.py` doesn't export the agent  
**Solution:** Ensure `your_agent_folder/__init__.py` has:
```python
from .agent import root_agent
__all__ = ["root_agent"]
```

---

### Agent deployed but doesn't respond / fails at runtime

**Checklist:**
1. ✅ Check logs in Cloud Console
2. ✅ Verify all dependencies are in `requirements`
3. ✅ Confirm `.env` uses Vertex AI:
   ```
   GOOGLE_GENAI_USE_VERTEXAI="True"
   GOOGLE_CLOUD_PROJECT="your-project-id"
   ```
4. ✅ Verify service account permissions:
   ```bash
   gcloud projects add-iam-policy-binding PROJECT_ID \
     --member="serviceAccount:service-PROJECT_NUMBER@gcp-sa-aiplatform-re.iam.gserviceaccount.com" \
     --role="roles/aiplatform.user"
   ```

---

## ✅ Pre-Deployment Checklist

Before deploying, verify:

- [ ] File structure is correct:
  ```
  your_project/
  ├── your_agent_folder/
  │   ├── __init__.py      # Exports root_agent
  │   ├── agent.py         # Defines root_agent
  │   └── .env             # Vertex AI configuration
  └── deploy_agent.py      # Deployment script
  ```

- [ ] `__init__.py` exports the agent:
  ```python
  from .agent import root_agent
  __all__ = ["root_agent"]
  ```

- [ ] `.env` uses Vertex AI:
  ```bash
  GOOGLE_GENAI_USE_VERTEXAI="True"
  GOOGLE_CLOUD_PROJECT="your-project-id"
  GOOGLE_CLOUD_LOCATION="us-central1"
  ```

- [ ] Credentials configured:
  ```bash
  gcloud auth application-default login
  ```

- [ ] Agent works locally:
  ```python
  from your_agent_folder.agent import root_agent
  response = root_agent.send_message("test")
  print(response)
  ```

- [ ] GCS bucket exists:
  ```bash
  gsutil ls gs://your-bucket
  ```

- [ ] IAM permissions configured (if necessary)

---

## 📚 Additional Resources

### Official Documentation
- [ADK Deployment Guide](https://google.github.io/adk-docs/deploy/agent-engine/)
- [Agent Engine Documentation](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine)
- [Troubleshooting Deploy](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/troubleshooting/deploy)

### Useful Commands

```bash
# View deployed agent logs
gcloud logging read "resource.type=aiplatform.googleapis.com/ReasoningEngine" \
  --project=PROJECT_ID --limit=50

# List deployed agents
gcloud ai reasoning-engines list --region=LOCATION

# Delete an agent
gcloud ai reasoning-engines delete RESOURCE_ID --region=LOCATION
```

---

## 🎯 Quick Summary

### ❌ DON'T DO THIS:
```bash
adk deploy agent_engine ...  # CLI has bugs
```

### ✅ DO THIS:
```python
# deploy_agent.py
from vertexai.agent_engines import AdkApp
from vertexai import agent_engines

app = AdkApp(agent=root_agent)
remote_app = agent_engines.create(
    app,
    requirements=["google-cloud-aiplatform[adk,agent_engines]>=1.132.0"],
    extra_packages=["./your_agent_folder"],  # ⭐ IMPORTANT
    display_name="My Agent",
)
```

### 🧪 TESTING:
```python
# Use async_stream_query, NOT query()
async for event in remote_app.async_stream_query(...):
    print(event)
```

---

**Last updated:** 2026-01-03  
**Contact:** Adri - Applied AI Engineer @ Aselvia
