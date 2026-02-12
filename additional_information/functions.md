---
parent: "Additional Information"
title: "PPI filter function"
nav_order: 3
---
## PPI filter function
In the [section on Open WebUI functions](../functions/quickstart_function.md) you saw a function for filtering PPI. Here, we give some background on PPI filtering and explain the code in detail.

### About PPI filtering
Personally Identifiable Information (PII) or Personal Data filtering is a critical component in any system that processes real-world user or employee information, especially when large language models or automated tools are involved. Without proper filtering, sensitive attributes such as names, email addresses, phone numbers, birth dates, or job roles can be inadvertently exposed, logged, or even leaked to downstream systems. A well-known example is the [2017 Equifax data breach](https://en.wikipedia.org/wiki/2017_Equifax_data_breach), where insufficient safeguards around access and data handling led to the exposure of highly sensitive personal information for over 140 million individuals, including Social Security numbers and birth dates. While that incident was not caused by an LLM, it illustrates how devastating improper handling of personal data can be, both legally and reputationally, and why modern AI toolchains must treat PPI as a first-class security concern.

In the context of MCP tool servers and LLM integrations, PPI filtering ensures that models only see or return information that is strictly necessary for the task. For instance, if a user asks for statistical counts of employees by role or gender, the system should never return raw emails or phone numbers, even if those fields exist in the underlying dataset.

### Code explanation
We begin by importing the required modules and defining a dictionary of compiled regex patterns. Each key is a human-readable label for the type of secret (e.g. `"API key"`), and each value is a regular expression that matches common formats of that secret. This design makes it straightforward to add new patterns later.
```python
import re
from pydantic import BaseModel
from typing import Optional, Set

SENSITIVE_PATTERNS = {
    "API key": re.compile(r"(api[_-]?key\s*[:=]\s*[A-Za-z0-9_\-]{16,})", re.IGNORECASE),
    "Private key": re.compile(
        r"-----BEGIN (?:RSA |EC |DSA )?PRIVATE KEY-----[\s\S]*?(?:-----END (?:RSA |EC |DSA )?PRIVATE KEY-----|$)",
        re.IGNORECASE,
    ),
    "AWS secret access key": re.compile(
        r"(aws[_-]?secret[_-]?access[_-]?key\s*[:=]\s*[A-Za-z0-9/+=]{40})",
        re.IGNORECASE,
    ),
}
```

Next we define the `Filter` class. Open WebUI expects a `Valves` inner class for user-configurable settings — here it's empty since no configuration is needed. The constructor initializes a set to track which categories of sensitive data were detected, a `toggle` flag that lets users enable or disable the filter in the UI, and a base64-encoded SVG icon displayed alongside the filter name.

```python
class Filter:
    class Valves(BaseModel):
        pass

    def __init__(self):
        self.valves = self.Valves()
        self.detected_categories: Set[str] = set()
        self.toggle = True
        self.icon = """data:image/svg+xml,%3C%3Fxml%20version%3D%221.0%22%20encoding%3D%22iso-8859-1%22%3F%3E%0A%3C!--%20Uploaded%20to%3A%20SVG%20Repo%2C%20www.svgrepo.com%2C%20Generator%3A%20SVG%20Repo%20Mixer%20Tools%20--%3E%0A%3C!DOCTYPE%20svg%20PUBLIC%20%22-%2F%2FW3C%2F%2FDTD%20SVG%201.1%2F%2FEN%22%20%22http%3A%2F%2Fwww.w3.org%2FGraphics%2FSVG%2F1.1%2FDTD%2Fsvg11.dtd%22%3E%0A%3Csvg%20fill%3D%22%23000000%22%20height%3D%22800px%22%20width%3D%22800px%22%20version%3D%221.1%22%20id%3D%22Capa_1%22%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20xmlns%3Axlink%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxlink%22%20%0A%09%20viewBox%3D%220%200%20347.971%20347.971%22%20xml%3Aspace%3D%22preserve%22%3E%0A%3Cpath%20d%3D%22M317.309%2C54.367C257.933%2C54.367%2C212.445%2C37.403%2C173.98%2C0C135.519%2C37.403%2C90.033%2C54.367%2C30.662%2C54.367%0A%09c0%2C97.405-20.155%2C236.937%2C143.317%2C293.604C337.463%2C291.305%2C317.309%2C151.773%2C317.309%2C54.367z%20M162.107%2C225.773l-47.749-47.756%0A%09l21.379-21.378l26.37%2C26.376l50.121-50.122l21.378%2C21.378L162.107%2C225.773z%22%2F%3E%0A%3C%2Fsvg%3E"""
```

The `scrub_pii` method does the actual redaction work. It loops through every pattern, checks for matches, records which categories were found, and replaces all occurrences with `[REDACTED]`.

```python
def scrub_pii(self, text: str) -> str:
    """Redact sensitive information from the text."""
    for label, pattern in SENSITIVE_PATTERNS.items():
        if pattern.search(text):
            self.detected_categories.add(label)
            text = re.sub(pattern, "[REDACTED]", text)
    return text
```

The `inlet` method intercepts the user's message **before** it reaches the LLM. It resets the detection tracker, emits a status event so the UI can show that filtering is active, and scrubs only the most recent message — earlier messages were already filtered on previous turns.

```python
async def inlet(
    self, body: dict, __event_emitter__, __user__: Optional[dict] = None
) -> dict:
    self.detected_categories = set()  # reset per request

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

    # Only filter the last user message (previous messages are already filtered)
    last_message = body["messages"][-1]["content"]
    body["messages"][-1]["content"] = self.scrub_pii(last_message)

    return body
```

The `outlet` method runs **after** the LLM has responded. If any sensitive categories were flagged during the inlet phase, it prepends a warning banner to the assistant's reply so the user knows redaction occurred. Otherwise it passes the response through unchanged.

```python
def outlet(self, body: dict, __user__: Optional[dict] = None) -> dict:
    if not self.detected_categories:
        return body

    categories = ", ".join(sorted(self.detected_categories))
    warning = (
        f"**⚠️ Sensitive information detected and redacted: {categories}.**\n\n"
    )

    body["messages"][-1]["content"] = warning + body["messages"][-1]["content"]

    return body
```
_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
