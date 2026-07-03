# Gemini Flash Lite & Vertex AI Integration Guide

This guide details the implementation of `flash-lite-test.py`, a script showcasing next-generation Generative AI workflows. It demonstrates how to utilize the new **`google-genai` SDK** backed by **Google Cloud Vertex AI** to achieve low-latency streaming text generation, custom safety filters, thinking level adjustments, and integration with **Google Search Grounding** tools for real-time information retrieval.

---

## 🔍 Code Walkthrough & Reference

The full implementation of the script can be found in `flash-lite-test.py`:

```python
from google import genai
from google.genai import types
import base64
import os

def calculate_sum_of_squares(a, b):
  # Helper function (not registered as a tool in this specific runner)
  return a + b*b + a*a

def generate():
  # 1. Initialize the GenAI Client with Vertex AI enablement
  client = genai.Client(
      vertexai=True,
      project="ninghai-ccai", # Specify your active GCP project ID
      location="global",
  )

  # 2. Configure model and prompt content
  model = "gemini-3.1-flash-lite-preview"
  contents = [
    types.Content(
      role="user",
      parts=[
        types.Part.from_text(text="summary the events from yesterday")
      ]
    ),
  ]

  # 3. Configure tools (Google Search Grounding)
  tools = [
    types.Tool(google_search=types.GoogleSearch()),
  ]

  # 4. Assemble safety settings and thinking configuration
  generate_content_config = types.GenerateContentConfig(
    temperature = 1,
    top_p = 0.95,
    max_output_tokens = 65535,
    safety_settings = [
      types.SafetySetting(category="HARM_CATEGORY_HATE_SPEECH", threshold="OFF"),
      types.SafetySetting(category="HARM_CATEGORY_DANGEROUS_CONTENT", threshold="OFF"),
      types.SafetySetting(category="HARM_CATEGORY_SEXUALLY_EXPLICIT", threshold="OFF"),
      types.SafetySetting(category="HARM_CATEGORY_HARASSMENT", threshold="OFF")
    ],
    tools = tools,
    thinking_config=types.ThinkingConfig(
      thinking_level="LOW", # Control model's pre-response thought process
    ),
  )

  # 5. Execute streaming API request
  for chunk in client.models.generate_content_stream(
    model = model,
    contents = contents,
    config = generate_content_config,
    ):
    if not chunk.candidates or not chunk.candidates[0].content or not chunk.candidates[0].content.parts:
        continue
    print(chunk.text, end="")

if __name__ == "__main__":
  generate()
```

---

## 🛠️ Deep Dive: Architectural Elements

### 1. `google-genai` Client on Vertex AI
Unlike legacy libraries, the new `google-genai` SDK offers a single unified interface for both Google AI (Gemini Developer API) and Vertex AI. By initializing with `vertexai=True`, the client routes all traffic through GCP's enterprise-grade infrastructure.
- **Project Scope:** `project="ninghai-ccai"` binds client interactions to a specific GCP Billing/Access boundary.
- **Location Scope:** `location="global"` routes requests dynamically, though you can specify regional endpoints (e.g., `us-central1`) if residency requirements dictate.

### 2. Google Search Grounding Tool
To combat hallucinations and obtain real-time information (such as summarizing events from "yesterday"), the model is equipped with Google Search Grounding.
```python
tools = [
  types.Tool(google_search=types.GoogleSearch()),
]
```
When this tool is attached:
- Gemini autonomously determines if it needs search results to fulfill the user's prompt.
- It generates and runs Google searches, parses the top results, integrates the facts into its reasoning, and responds with inline citations.
- Grounding results can also return metadata with the specific sources and search suggestions.

### 3. Safety Settings Configuration
Safety thresholds are explicitly tuned via the `GenerateContentConfig` block. In development environments where raw responses are analyzed, thresholds can be softened or toggled entirely off (`OFF`):
- `HARM_CATEGORY_HATE_SPEECH`
- `HARM_CATEGORY_DANGEROUS_CONTENT`
- `HARM_CATEGORY_SEXUALLY_EXPLICIT`
- `HARM_CATEGORY_HARASSMENT`

Other threshold levels include `BLOCK_LOW_AND_ABOVE`, `BLOCK_MEDIUM_AND_ABOVE`, and `BLOCK_ONLY_HIGH`.

### 4. Thinking Configuration (`ThinkingConfig`)
For Gemini 3.1 models, **Reasoning/Thinking** is supported natively. This allows the model to output a hidden/explicit "thought process" chain prior to writing the final response.
```python
thinking_config=types.ThinkingConfig(
    thinking_level="LOW",
)
```
- **`thinking_level` Options:**
  - `OFF`: Disables reasoning entirely for fastest time-to-first-token.
  - `LOW`: Enables lightweight reasoning (best for low-latency tasks that still require a structured approach).
  - `MEDIUM` / `HIGH` / `DYNAMIC`: Controls the depth of the pre-response reasoning block.

### 5. Efficient Low-Latency Streaming
Instead of waiting for a long, comprehensive response to generate on the server, the script uses `client.models.generate_content_stream`. This yields chunks in real-time as they are ready:
- Loops through the generator.
- Robust checks (`not chunk.candidates or ...`) ensure the program won't crash if safety blocks trigger or empty chunks are encountered.
- Outputs stream chunks natively using `print(chunk.text, end="")`.
