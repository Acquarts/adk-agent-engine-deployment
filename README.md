# ADK Agent Engine Deployment

> Production-ready deployment solution for Google ADK Agents to Vertex AI Agent Engine

[![Python](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-apache2.0-green.svg)](LICENSE)


---

## 🎯 The Problem

The ADK CLI `adk deploy agent_engine` has a critical bug that causes deployments to fail with:

```
ModuleNotFoundError: No module named 'agent_engine_app'
```

**This repository provides the solution.**

---

## ✅ The Solution

This toolkit provides **programmatic deployment templates** that bypass CLI bugs and ensure reliable deployments every time.

### Why This Works

| CLI Approach ❌ | Programmatic Approach ✅ |
|----------------|-------------------------|
| Creates temporary files with import bugs | Uses Python API directly |
| Implicit package handling | Explicit `extra_packages` control |
| Runs problematic `absolufy-imports` | Avoids intermediate steps |
| Limited error handling | Comprehensive error handling |

---

## 🚀 Quick Start

### 1. Copy Templates to Your Project

```bash
# English templates
cp templates/deploy_agent.py your_project/
cp templates/test_agent.py your_project/

# Spanish templates  
cp templates/deploy_agent.py your_project/  # (versión en español)
cp templates/test_agent.py your_project/
```

### 2. Configure and Deploy

```python
# Edit deploy_agent.py
PROJECT_ID = "your-project-id"
AGENT_FOLDER = "your_agent_folder"
# ... other variables

# Deploy
python deploy_agent.py
```

### 3. Test

```python
# Edit test_agent.py  
TEST_MESSAGES = ["Hello!", "What can you do?"]

# Test
python test_agent.py
```

**That's it!** ✨

---

## 📚 Documentation

### 📖 Documentation
- **[Complete Deployment Guide](docs/en/ADK_DEPLOYMENT_GUIDE.md)** - Detailed explanations and examples
- **[Quick Reference Cheatsheet](docs/en/ADK_DEPLOYMENT_CHEATSHEET.md)** - Fast lookup
- **[README](docs/en/README.md)** - How to use the templates


---

## 🛠️ Features

- ✅ **Bypasses ADK CLI bugs** - No more `ModuleNotFoundError`
- ✅ **Production-ready templates** - Copy, configure, deploy
- ✅ **Comprehensive error handling** - Clear error messages
- ✅ **Automated testing scripts** - Verify deployments work
- ✅ **Bilingual documentation** - English & Spanish
- ✅ **Battle-tested** - Used in production environments

---

## 📁 Repository Structure

```
adk-agent-engine-deployment/
├── README.md                          # This file
├── LICENSE                            # MIT License
├── docs/
│   ├── en/                           # English documentation
│   │   ├── README.md
│   │   ├── ADK_DEPLOYMENT_GUIDE.md
│   │   └── ADK_DEPLOYMENT_CHEATSHEET.md
│   └── es/                           # Spanish documentation
│       ├── README.md
│       ├── ADK_DEPLOYMENT_GUIDE.md
│       └── ADK_DEPLOYMENT_CHEATSHEET.md
├── templates/
│   ├── deploy_agent.py               # Deployment template
│   └── test_agent.py                 # Testing template
└── examples/
    └── weather_agent/                # Complete working example
        ├── agent.py
        ├── __init__.py
        ├── deploy_agent.py
        └── test_agent.py
```

---

## 🎯 How It Works

### The CLI Problem

```bash
# ❌ This fails
adk deploy agent_engine --project=... --region=... agent_folder

# Error: ModuleNotFoundError: No module named 'agent_engine_app'
```

### Our Solution

```python
# ✅ This works
from vertexai.agent_engines import AdkApp
from vertexai import agent_engines

app = AdkApp(agent=root_agent)
remote_app = agent_engines.create(
    app,
    requirements=["google-cloud-aiplatform[adk,agent_engines]>=1.132.0"],
    extra_packages=["./your_agent_folder"],  # ⭐ Key difference
    display_name="My Agent",
)
```

**Key Points:**
1. Uses Python API directly (no CLI)
2. Explicitly includes agent folder via `extra_packages`
3. Avoids temporary file creation bugs
4. Full control over deployment process

---

## 📖 Usage Example

```python
# 1. Create your agent
from google.adk.agents import Agent

root_agent = Agent(
    name="my_agent",
    model="gemini-2.0-flash",
    instruction="You are a helpful agent",
    tools=[get_weather, get_time]
)

# 2. Copy templates
cp templates/deploy_agent.py .
cp templates/test_agent.py .

# 3. Configure
# Edit deploy_agent.py with your PROJECT_ID, etc.

# 4. Deploy
python deploy_agent.py

# 5. Test  
python test_agent.py
```

---

## 🔧 Troubleshooting

### Common Errors

| Error | Solution |
|-------|----------|
| `ModuleNotFoundError: No module named 'agent_engine_app'` | Don't use CLI, use our templates |
| `'AgentEngine' object has no attribute 'query'` | Use `async_stream_query()` not `query()` |
| `No module named 'your_agent_folder'` | Add `extra_packages=["./your_agent_folder"]` |
| `cannot import name 'root_agent'` | Verify `__init__.py` exports `root_agent` |

**Full troubleshooting guide:** [English](docs/en/ADK_DEPLOYMENT_GUIDE.md#troubleshooting) | [Español](docs/es/ADK_DEPLOYMENT_GUIDE.md#troubleshooting)

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

### Areas for Contribution
- Additional examples
- Support for other deployment targets (Cloud Run, GKE)
- Automated testing improvements
- Documentation translations to other languages

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👨‍💻 Author

**Adri** - Applied AI Engineer @ Aselvia

- Expertise: ADK, Multi-Agent Systems, Google Cloud
- Background: 3D Video Game Artist turned AI Engineer

---

## 🙏 Acknowledgments

- Google ADK team for the excellent framework
- Vertex AI Agent Engine team
- Community members who identified and reported the CLI bug

---

## 🔗 Useful Links

- [Google ADK Documentation](https://google.github.io/adk-docs/)
- [Vertex AI Agent Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine)
- [ADK GitHub Repository](https://github.com/google/adk-python)

---

## ⭐ Star This Repository

If this toolkit helped you deploy your ADK agents successfully, please consider giving it a star! ⭐

It helps others discover this solution and motivates continued maintenance.

---

**Quick Links:**
- [📖 Docs](docs/en/README.md)
- [🚀 Quick Start](#quick-start)
- [🐛 Report Bug](https://github.com/yourusername/adk-agent-engine-deployment/issues)
