# ADK Agent Engine - Cheat Sheet 🚀

## ❌ Problem: CLI fails with `ModuleNotFoundError: No module named 'agent_engine_app'`

## ✅ Solution: Programmatic Deployment

### 1. Minimal Deployment Template

```python
import vertexai
from vertexai.agent_engines import AdkApp
from vertexai import agent_engines

vertexai.init(
    project="PROJECT_ID",
    location="us-central1",
    staging_bucket="gs://BUCKET"
)

from your_agent_folder.agent import root_agent

app = AdkApp(agent=root_agent)
remote_app = agent_engines.create(
    app,
    requirements=["google-cloud-aiplatform[adk,agent_engines]>=1.132.0"],
    extra_packages=["./your_agent_folder"],  # ⭐ CRITICAL
    display_name="My Agent",
)

print(f"✅ Deployed: {remote_app.resource_name}")
```

### 2. Testing Template

```python
import asyncio
import vertexai
from vertexai import agent_engines

vertexai.init(project="PROJECT_ID", location="us-central1")

async def test():
    remote_app = agent_engines.get("RESOURCE_NAME")
    session = await remote_app.async_create_session(user_id="user_123")
    
    async for event in remote_app.async_stream_query(
        user_id="user_123",
        session_id=session["id"],
        message="Hello!"
    ):
        print(event)

asyncio.run(test())
```

### 3. Required File Structure

```
project/
├── your_agent_folder/
│   ├── __init__.py      # from .agent import root_agent
│   ├── agent.py         # Defines root_agent
│   └── .env             # GOOGLE_GENAI_USE_VERTEXAI="True"
└── deploy_agent.py
```

### 4. Correct Methods

```python
# ❌ DON'T EXIST
remote_app.query()
remote_app.send_message()

# ✅ USE THESE (ASYNC)
await remote_app.async_create_session(user_id)
await remote_app.async_stream_query(user_id, session_id, message)
```

### 5. Useful Commands

```bash
# View logs
gcloud logging read "resource.type=aiplatform.googleapis.com/ReasoningEngine" --project=PROJECT_ID --limit=50

# List agents
gcloud ai reasoning-engines list --region=us-central1

# Permissions (if failing due to permissions)
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:service-PROJECT_NUMBER@gcp-sa-aiplatform-re.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"
```

### 6. Quick Troubleshooting

| Error | Solution |
|-------|----------|
| `No module named 'agent_engine_app'` | Use API instead of CLI |
| `'AgentEngine' object has no attribute 'query'` | Use `async_stream_query()` |
| `No module named 'your_agent_folder'` | Add `extra_packages=["./your_agent_folder"]` |
| `cannot import name 'root_agent'` | Verify `__init__.py` exports root_agent |

---

**TL;DR:**
1. DON'T use `adk deploy agent_engine`
2. USE `agent_engines.create()` with `extra_packages`
3. Testing with `async_stream_query()`, NOT `query()`
