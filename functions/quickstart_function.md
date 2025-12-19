---
title: "Functions in Open WebUI"
parent: "Functions and Tools"
nav_order: 1
---

## What are functions?
Functions allow you to extend the capabilities of a language model by defining reusable, server-side logic that the model can call when appropriate. Open WebUI functions are a lightweight and tightly integrated mechanism: they live directly inside your Open WebUI instance and are written as small Python snippets. The full documentation for functions is available [here](https://docs.openwebui.com/features/plugin/functions/). 

There are three types of functions you can use:
- **Pipe functions:** With a pipe function, you create your own custom models or agents. With a pipe you can set up a complex workflow, potentially combining different models. The pipe function will appear as a separate model in the UI. 
- **Filter function:** With a filter function, you can modify the input before it gets send to the model, or modify the output before it is returned to the user. The filter function can be enabled for specific models, or globally.
- **Action function:** With an action function, you create custom buttons underneat the chat interface, with which users can trigger specific actions. 

In this section, you will learn how to add functions to Open WebUI, and how to enable them in a chat. In the next section, you will build your own Open WebUI function for an agentic coding framework. 

## Filter functions
Each of the different function types has it's own structure, which is described in the [documentation](https://docs.openwebui.com/features/plugin/functions/). Here, we will discuss the [filter function](https://docs.openwebui.com/features/plugin/functions/filter), while in the next section you will work with pipe functions. The filter function is structured as follows:

```python
from pydantic import BaseModel
from typing import Optional

class Filter:
    # Valves: Configuration options for the filter
    class Valves(BaseModel):
        pass

    def __init__(self):
        # Initialize valves (optional configuration for the Filter)
        self.valves = self.Valves()

    def inlet(self, body: dict) -> dict:
        # This is where you manipulate user inputs.
        print(f"inlet called: {body}")
        return body

    def stream(self, event: dict) -> dict:
        # This is where you modify streamed chunks of model output.
        print(f"stream event: {event}")
        return event

    def outlet(self, body: dict) -> None:
        # This is where you manipulate model outputs.
        print(f"outlet called: {body}")
```

A filter contains the following components:
* **Valves:** Valves are the settings of a filter, that a user can configure.
* **Inlet:** the inlet function allows you to modify the user input, for example adding more context or sanitiziation.
* **Stream:** the stream function intercepts individual chunks of the model output, allowing you to modify them in real-time.
* **Outlet:** the outlet function is the last part of the pipeline, enabling you to adjust the final LLM output. 

## Adding your function to Open WebUI
We will now experiment with an existing filter function, obtained from the community-created functions available [here](https://openwebui.com/?sort=new&t=all). 



{: .action}
1. In your Open WebUI instance, go to `admin panel` -> `functions`
2. In the top right, click on `+ New Function`
3.  

## Writing your own function
