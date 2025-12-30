---
title: "Additional Exercises"
parent: "Functions and Agentic Workflows"
nav_order: 3
---

## Additional exercises
In the previous sections, you learned how to:
- Use **filter functions** to intercept and modify user input and model output
- Build **pipe functions** to construct agentic workflows with multiple LLM calls

In this final part of the workshop, you will deepen your understanding by:
- Exploring **action functions**
- Using **valves** as user-configurable controls
- Extending agentic pipelines with additional reasoning steps
- Reflecting on design trade-offs in agentic systems

You do not need to complete all of them—focus on the ones that best match your interests and available time. For exercise 1 and 3, a solution is available.

## Exercise 1: Building an Action Function
[Action Functions](https://docs.openwebui.com/features/plugin/functions/action/) will appear as clickable buttons below the generated response from the LLM. Whereas filter functions automatically execute operations on the input prompt or generated response, action functions need to be manually triggered by a user by clicking on the button below the response from the LLM. Several examples of action functions are:

* Summarizing the response from the LLM (_this exercise_)
* [Visualizing a response from a LLM](https://openwebui.com/posts/4319b6c2-5070-43e4-8310-738cd70ae61f)
* [Convert LLM responses in markdown to Word documents](https://openwebui.com/posts/76716d12-5896-4698-a4ac-62fa23dd7251)

The template of an action function is as follows:

```python
class Action:
    def __init__(self):
        self.valves = self.Valves()

    class Valves(BaseModel):
        # Configuration parameters
        parameter_name: str = "default_value"

    async def action(self, body: dict, __user__=None, __event_emitter__=None, __event_call__=None):
        # Action implementation
        return {"content": "Modified message content"}
```

## Exercise 2: Adding Valves to Your Filters

In Part 1, the PPI filter always redacted detected sensitive information. In practice, users may want more control.

### Goal
Extend the PPI filter with one or more **valves**, such as:
- A toggle to enable/disable redaction
- A “warn only” mode (detect but do not redact)
- A verbosity level for user warnings

Example ideas:
- `MODE = "redact" | "warn"
- `SHOW_WARNING_MESSAGE: bool`

### Reflection
- How do valves turn a function into a reusable *tool* rather than a hard-coded rule?
- Which settings should be exposed to users, and which should remain internal?

---

## Exercise 3: Extending the SafeCoder Agentic Pipeline
