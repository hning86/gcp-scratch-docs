# RAG Traps

This document outlines common "RAG traps" that can occur when writing asynchronous code. These are patterns that may appear to work correctly but can lead to performance issues or unexpected behavior.

The following file, `demo_rag_traps.py`, was recently added to the `hning86/gcp-scratch` repository and contains several of these traps:

```python
import time
import requests

async def handle_incoming_webhook(payload: dict):
    # RAG Violation 1: Synchronous sleep inside async function
    time.sleep(5)

    try:
        # RAG Violation 2: Synchronous requests.get inside async function without asyncio.to_thread
        resp = requests.get("https://api.github.com/zen")
        return resp.text
    except Exception:
        # RAG Violation 3: Bare except without exc_info=True or logging
        pass
```

## Violations

### 1. Synchronous sleep inside async function

The `time.sleep(5)` call is a blocking operation. In an asynchronous function, this will block the entire event loop, preventing any other concurrent tasks from running. The correct way to sleep in an async function is to use `await asyncio.sleep(5)`.

### 2. Synchronous `requests.get` inside async function

The `requests.get()` call is also a blocking I/O operation. It will block the event loop while waiting for the network request to complete. To avoid this, you should use an async HTTP library like `aiohttp` or run the synchronous call in a separate thread using `asyncio.to_thread`.

### 3. Bare `except`

The `except Exception:` clause catches all exceptions, including things like `KeyboardInterrupt` and `SystemExit`. This can make it difficult to debug issues and can hide bugs. It is better to catch more specific exceptions. Additionally, the bare `pass` means that the exception is silently ignored, which is almost never a good idea. If you must catch a broad exception, you should at least log it.
