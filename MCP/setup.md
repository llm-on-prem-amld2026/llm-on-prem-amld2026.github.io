---
title: "Setting up your tool server"
parent: "Model Context Protocol (MCP)"
nav_order: 1
---

**Model Context Protocol (MCP)** is an open standard that defines how AI models interact with external tools and systems. Instead of relying on prompt hacks or application-specific integrations, MCP provides a structured, machine-readable way to expose capabilities such as data access, document retrieval, or system queries to a model. 

In this workshop, we will run a small MCP server and connect it to Open WebUI, giving the model controlled access to a useful capabilities. You will see how tools are discovered, how the model decides to call them, and how structured results flow back into the conversation. The goal is to understand how MCP enables reliable, reusable tool use and forms the foundation for building real agents.

## Setting up Python on your machine

## Launching your tool server
```python
from fastmcp import FastMCP
import requests
mcp = FastMCP("useful-tools-demo", port=8000)
mcp.settings.host = "0.0.0.0"

@mcp.tool
def get_weather(city: str):
    """Returns the current weather for a city."""
    try:
        response = requests.get(f"https://wttr.in/{city}?format=j1")
        data = response.json()
        current = data["current_condition"][0]
        return {
            "city": city,
            "temperature_C": current["temp_C"],
            "temperature_F": current["temp_F"],
            "weather": current["weatherDesc"][0]["value"],
            "humidity": current["humidity"],
        }
    except Exception as e:
        return {"error": str(e)}

if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```

## Accessing the tools through Open WebUI
