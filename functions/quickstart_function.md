---
title: "Functions in Open WebUI"
parent: "Functions and Tools"
nav_order: 1
---

## What are functions?
Functions allow you to extend the capabilities of a language model by defining reusable, server-side logic that the model can call when appropriate. Open WebUI functions are a lightweight and tightly integrated mechanism: they live directly inside your Open WebUI instance and are written as small Python snippets.

Functions are useful when you want to:
- Add custom logic without running a separate server
- Post-process or validate model outputs
- Expose internal APIs, scripts, or utilities to the model
- Prototype agent-like behavior quickly

In this section, you will learn how functions work conceptually, how to add them to Open WebUI, and how to enable them in a chat. In the next section, you will build your own Open WebUI function to 

## How functions work
At a high level, functions in Open WebUI follow this flow:
