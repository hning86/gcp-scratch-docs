# Google Cloud Platform (GCP) & Multi-Agent Workflows Documentation

Welcome to the documentation for the GCP Scratch project. This repository contains a collection of Python scripts demonstrating various Google Cloud services, Generative AI (GenAI) integration, Natural Language Processing (NLP), and multi-agent workflow orchestration using the Google Agent Development Kit (ADK).

This documentation is organized into topic-specific subdirectories to provide targeted, in-depth guides for each implementation.

---

## 📂 Documentation Directory

To explore specific topics and code implementation walkthroughs, navigate to the following guides:

### 1. 🤖 Generative AI & Vertex AI
- **[Gemini Flash Lite & Vertex AI Integration](docs/genai/gemini_flash_lite.md)**
  - Explores `flash-lite-test.py`.
  - Topics: Streaming response generation with `gemini-3.1-flash-lite-preview`, Vertex AI API backend integration, **Google Search Grounding** as a tool, safety threshold configurations, and low-latency thinking configurations (`ThinkingConfig`).

### 2. 📝 Natural Language Processing (NLP)
- **[Named Entity Recognition (NER) with Cloud Natural Language API](docs/nlp/natural_language_ner.md)**
  - Explores `ner.py`.
  - Topics: Text entity analysis, salience scores, language auto-detection, and extracting entity metadata (e.g., Wikipedia URLs) and text mentions using the `google-cloud-language` SDK.

### 3. 👥 Multi-Agent Workflows (Google ADK)
- **[Asynchronous, Non-Blocking Agents](docs/agents/adk_long_running.md)**
  - Explores `long_running_agent/agent.py` and `long_running.py`.
  - Topics: Non-blocking, long-running agent orchestration, background tasks using daemon threads, filesystem state-tracking, and the **Root ➔ Starter Sub-Agent ➔ Checker Sub-Agent** design pattern.

- **[Deterministic Agent Chaining & StoryFlow](docs/agents/adk_storyflow.md)**
  - Explores `storyflow.py`.
  - Topics: Multi-agent sequential processing, feedback loop orchestration (`LoopAgent`), post-processing checks (`SequentialAgent`), tone-based conditional logic, session state preservation, and final artifact generation.

---

## 🛠️ Environment Prerequisites & Setup

### 1. Google Cloud Configuration
To run any of the scripts in this repository, you must have a Google Cloud Project with the required APIs enabled and local application default credentials configured.

*   **API Enablement:** Enable the following APIs in your Google Cloud Console:
    *   Vertex AI API
    *   Cloud Natural Language API
*   **Authentication (ADC):** Set up local authentication credentials:
    ```bash
    gcloud auth application-default login
    ```
*   **Project Identification:** Set your active Google Cloud project ID. For example, in `flash-lite-test.py`, ensure the client initialization points to your active project ID (e.g., `"ninghai-ccai"`).

### 2. Installation using `uv`
This project uses the modern, lightning-fast Python package installer and resolver `uv`.

*   **Install Dependencies:** Run the following command from the root of the `gcp-scratch` directory to synchronize the environment and lockfiles:
    ```bash
    uv sync
    ```

*   **Dependencies Summary:**
    *   `google-adk`: Google's Agent Development Kit for building agent workflows.
    *   `google-genai`: Google's next-generation client library for Gemini models.
    *   `google-cloud-language`: Client library for the Cloud Natural Language API.
    *   `pyopenssl`: Core dependency supporting secure SSL communication.

---

## 🚀 Running the Scripts

Execute the scripts using `uv` from the `gcp-scratch` project directory:

```bash
# Run the Gemini Flash Lite Streaming & Search Grounding script
uv run flash-lite-test.py

# Run the Named Entity Recognition analyzer
uv run ner.py

# Run the StoryFlow deterministic narrative orchestrator
uv run storyflow.py

# Run the long-running agent demo (CLI version)
uv run long_running.py

# Run the long-running agent web-based chat interface (ADK-driven)
uv run adk web long_running_agent
```
