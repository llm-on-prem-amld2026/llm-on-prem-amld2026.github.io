---
parent: "Additional Information"
title: "Open WebUI and Ollama"
nav_order: 2
---

## Open WebUI and Ollama

**Open WebUI** is a web-based user interface for interacting with large language models. It provides a simple, browser-accessible front end where you can send prompts, manage models, and configure settings without needing to work directly on the command line. In this workshop, Open WebUI serves as the main interface through which you will interact with the LLMs running on your compute instance, while the actual model execution happens in the background. Here you can find the [official Open WebUI documentation](https://docs.openwebui.com/getting-started/).

**Ollama** is a lightweight runtime that manages and runs large language models locally. It handles tasks such as downloading models, loading them into memory, and exposing an API that other applications—like Open WebUI—can communicate with. By running Ollama inside Docker with GPU support enabled, we can efficiently serve LLMs on the compute instance while keeping the setup reproducible and easy to manage. Here you can find the [official Ollama documentation](https://docs.ollama.com/).

_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
