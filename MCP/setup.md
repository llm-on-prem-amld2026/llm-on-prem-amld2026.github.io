---
title: "Setting up your tool server"
parent: "Model Context Protocol (MCP)"
nav_order: 1
---

**Model Context Protocol (MCP)** is an open standard that defines how AI models interact with external tools and systems. Instead of relying on prompt hacks or application-specific integrations, MCP provides a structured, machine-readable way to expose capabilities such as data access, document retrieval, or system queries to a model. 

In this workshop, we will run a small MCP server and connect it to Open WebUI, giving the model controlled access to a useful capabilities. You will see how tools are discovered, how the model decides to call them, and how structured results flow back into the conversation. The goal is to understand how MCP enables reliable, reusable tool use and forms the foundation for building real agents.

## Setting up the environment
We will start by preparing a Python environment using `uv` and a virtual environment (`venv`).  

{: .action}
> 1. Install the `uv` environment manager:  
> ```bash
> curl -LsSf https://astral.sh/uv/install.sh | sh
> source $HOME/.local/bin/env
> uv init tools_server
> cd tools_server
> ```  
> 2. Create and activate a virtual environment:  
> ```bash
> uv venv
> source .venv/bin/activate
> ```  
> 3. Ensure `pip` is installed and updated:  
> ```bash
> sudo apt-get update
> sudo apt install python3-pip
> python -m ensurepip --upgrade
> python -m pip install fastmcp
> ```

This sets up an isolated Python environment where we can safely install dependencies without affecting the system Python.  

## Launching the tool server
Next, we will write a simple Python script that creates an MCP server and exposes a `get_weather` tool.  

{: .action}
> 1. Open a new Python file with `nano tools.py`
> 2. Copy the code below into the file, and exit and save through `ctrl+X` followed by `y`
> 3. Run your tool server through the command `python tools.py`

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

## Configuring the tools in Open WebUI
Next, we will connect our tool server to Open WebUI. Follow the steps below to achieve this.

{: .action}
> 1. In Open WebUI, go to `Admin panel -> External Tools -> Add Connection`
> 2. Fill in the fields as follows:
>     * **Type**: MCP
>     * **URL**: `http://IP:8000/mcp` (replace `IP` with your server's IP address)
>     * **Headers**: `{"Accept": "application/json, text/event-stream", "Content-Type": "application/json"}`
>     * **ID**: `tools`
>     * **Name/Description**: Here you can give the tool server a name and a description
>     * **Function Name Filter List**: 

## Using the tool in Open WebUI

## What's next?
