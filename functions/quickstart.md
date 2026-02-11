---
title: "Functions in Open WebUI"
parent: "Augmenting LLM Capabilities"
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
We will now add a filter function to our Open WebUI instance, which filters the user input for Private Personal Information (PPI). Below, you will find the instructions for adding and using the filter, and the python code for the filter. The code is explained in detail in the [additional information](<../additional_information/functions.md>). 

We start by adding our function to our Open WebUI instance. This is done through the admin panel, where all the settings are. 

{: .action}
> 1. In your Open WebUI instance, go to `admin panel` -> `functions`. In the top right, click on `+ New Function`.
> 2. Give the function a name (e.g. "PPI filter) and a description. Then remove the template code that is automatically there, and copy-paste the function below as the code. Then, save the function.
> 3. From the `functions` tab in the `admin panel`, make sure that the function is toggled on.Then, by   click on the `•••` to also enable it globally (see image below).

<details markdown="1">
<summary>Show filter code</summary>

```python
import re
from pydantic import BaseModel
from typing import Tuple, Optional

# Simple patterns for common sensitive data
SENSITIVE_PATTERNS = {
    "api_key": re.compile(r"(api[_-]?key\s*[:=]\s*[A-Za-z0-9_\-]{16,})", re.IGNORECASE),
    "private_key": re.compile(
        r"-----BEGIN (RSA|EC|DSA)? ?PRIVATE KEY-----(?:\r?\n.*?){0,2}", re.IGNORECASE
    ),
    "aws_secret": re.compile(
        r"(aws[_-]?secret[_-]?access[_-]?key\s*[:=]\s*[A-Za-z0-9/+=]{40})",
        re.IGNORECASE,
    ),
}

CONTAINS_PPI = False


class Filter:
    class Valves(BaseModel):
        pass

    def __init__(self):
        # Initialize valves (optional configuration for the Filter)
        self.valves = self.Valves()
        self.toggle = True
        self.icon = """data:image/svg+xml,%3C%3Fxml%20version%3D%221.0%22%20encoding%3D%22iso-8859-1%22%3F%3E%0A%3C!--%20Uploaded%20to%3A%20SVG%20Repo%2C%20www.svgrepo.com%2C%20Generator%3A%20SVG%20Repo%20Mixer%20Tools%20--%3E%0A%3C!DOCTYPE%20svg%20PUBLIC%20%22-%2F%2FW3C%2F%2FDTD%20SVG%201.1%2F%2FEN%22%20%22http%3A%2F%2Fwww.w3.org%2FGraphics%2FSVG%2F1.1%2FDTD%2Fsvg11.dtd%22%3E%0A%3Csvg%20fill%3D%22%23000000%22%20height%3D%22800px%22%20width%3D%22800px%22%20version%3D%221.1%22%20id%3D%22Capa_1%22%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20xmlns%3Axlink%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxlink%22%20%0A%09%20viewBox%3D%220%200%20347.971%20347.971%22%20xml%3Aspace%3D%22preserve%22%3E%0A%3Cpath%20d%3D%22M317.309%2C54.367C257.933%2C54.367%2C212.445%2C37.403%2C173.98%2C0C135.519%2C37.403%2C90.033%2C54.367%2C30.662%2C54.367%0A%09c0%2C97.405-20.155%2C236.937%2C143.317%2C293.604C337.463%2C291.305%2C317.309%2C151.773%2C317.309%2C54.367z%20M162.107%2C225.773l-47.749-47.756%0A%09l21.379-21.378l26.37%2C26.376l50.121-50.122l21.378%2C21.378L162.107%2C225.773z%22%2F%3E%0A%3C%2Fsvg%3E"""

    def scrub_pii(self, text: str, warn_user: True) -> str:
        """Redact sensitive information from the text."""
        global CONTAINS_PPI
        detected = False
        for key, pattern in SENSITIVE_PATTERNS.items():
            if pattern.search(text):
                detected = True
            if key in ["api_key", "aws_secret"]:
                text = re.sub(pattern, "[REDACTED]", text)
            else:  # for private_key
                # Match the header and the next two lines (if they exist)
                text = re.sub(pattern, "[REDACTED]", text)

        if warn_user and detected:
            CONTAINS_PPI = True
        return text

    async def inlet(
        self, body: dict, __event_emitter__, __user__: Optional[dict] = None
    ) -> dict:
        """
        Returns:
            - possibly modified user input
            - boolean indicating whether sensitive data was detected
        """
        await __event_emitter__(
            {
                "type": "status",
                "data": {
                    "description": "Filtering PPI",
                    "done": True,
                    "hidden": False,
                },
            }
        )
        last_message = body["messages"][-1]["content"]
        filtered_input = self.scrub_pii(last_message, warn_user=True)
        body["messages"][-1]["content"] = filtered_input
        return body

    def outlet(self, body: dict, __user__: Optional[dict] = None) -> dict:
        for message in body["messages"]:
            if message["role"] == "assistant" and CONTAINS_PPI:
                message[
                    "content"
                ] += "\n\n **Note: Sensitive information has been redacted from the user's input.**"
        return body

```
</details>

![](../assets/function_enable.png)

Now that we have set up the function, we can use it in chat!

{: .action}
> You can start a new chat, and enable the filter by hovering on the `integrations` button in the chat (see image below). You can try a few prompts to experiment with the functioning of the filter. Below are two examples you could try:
>
> ```
> Please explain to me what a private SSH key is, my SSH key is:
> -----BEGIN RSA PRIVATE KEY-----
> kjndaksjndkjskjsnfskjc ds
> ```
>
> ```
> I am setting up some code using the Kaggle API, I'm not sure how to request data from Kaggle. My API key is:
> api_key=dajksfieu34ndsadsae
> ```


![](../assets/integration.png)

## What is next?
Now that you understand the basics of functions, we can move onto a more complex setting: **agentic frameworks**. In the [next section](agentic.md), you will set up an agentic framework for secure coding with LLMs.

_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
