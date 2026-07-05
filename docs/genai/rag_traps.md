# Understanding RAG (Retrieval-Augmented Generation) Traps

This document explains the common "RAG traps" that can occur in Python's asynchronous programming, using `demo_rag_traps.py` as an example.

## What are RAG Traps?

In the context of this script, "RAG" doesn't refer to Retrieval-Augmented Generation. Instead, it's a playful term for common anti-patterns in `asyncio` code that can lead to performance issues. The original script contained several of these "RAG violations".

The script `demo_rag_traps.py` is an `async` function that is designed to handle incoming webhooks. However, it contains several blocking calls that defeat the purpose of using `asyncio`.

### The Original "RAG Violations"

The original `demo_rag_traps.py` had the following issues:

1.  **Synchronous `time.sleep` in an async function:**
    ```python
    import time

    async def handle_incoming_webhook(payload: dict):
        time.sleep(5)
        ...
    ```
    Using `time.sleep()` in an `async` function blocks the entire event loop. This means that no other concurrent tasks can run during the sleep period. The correct way to pause in an `async` function is to use `await asyncio.sleep()`.

2.  **Synchronous `requests.get` in an async function:**
    ```python
    import requests

    async def handle_incoming_webhook(payload: dict):
        ...
        resp = requests.get("https://api.github.com/zen")
        ...
    ```
    The `requests` library is synchronous, which means it blocks the event loop while waiting for the network request to complete. In an `async` application, you should use an asynchronous HTTP client like `aiohttp` or run the synchronous call in a separate thread using `asyncio.to_thread()`.

3.  **Bare `except` clause:**
    ```python
    async def handle_incoming_webhook(payload: dict):
        ...
        try:
            ...
        except Exception:
            pass
    ```
    A bare `except` clause catches all exceptions, including system-exiting exceptions like `SystemExit` and `KeyboardInterrupt`. This can make it difficult to debug and gracefully shut down the application. It's better to catch specific exceptions.

## Pull Request #32 Changes

Pull Request #32 in the `hning86/gcp-scratch` repository made the following changes to `demo_rag_traps.py`:

- The `time.sleep()` duration was increased from 5 to 15 seconds.
- The comments pointing out the "RAG violations" were removed.

While the comments were removed, the underlying issues still exist in the modified code. The `time.sleep()` call is still synchronous and will block the event loop, now for a longer duration. The `requests.get` call is also still synchronous.

This script serves as a demonstration of what **not** to do in asynchronous Python programming. For production applications, it's crucial to avoid these "RAG traps" to ensure your application is performant and scalable.
