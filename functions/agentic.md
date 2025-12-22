---
title: "Agentic workflows"
parent: "Functions and Agentic Workflows"
nav_order: 2
---

We now move to a more advanced functionality in Open WebUI: [pipe functions](https://docs.openwebui.com/features/plugin/functions/pipe/). With a pipe, you build your own model in Open WebUI, with full control over how the data flows and which models are included in the pipeline. The basic structure of a pipe function is as follows:

```python
from pydantic import BaseModel, Field

class Pipe:
    class Valves(BaseModel):
        MODEL_ID: str = Field(default="")

    def __init__(self):
        self.valves = self.Valves()

    def pipes(self):
        return [
            {"id": "model_id_1", "name": "model_1"},
            {"id": "model_id_2", "name": "model_2"},
            {"id": "model_id_3", "name": "model_3"},
        ]

    def pipe(self, body: dict):
        # Logic goes here
        print(self.valves, body)  # Prints the configuration options and the input body
        model = body.get("model", "")
        return f"{model}: Hello, World!"
```
