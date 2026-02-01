---
title: "Additional Exercises"
parent: "Augmenting LLM Capabilities"
nav_order: 3
---

## Additional exercises
In the previous sections, you learned how to:
- Use **filter functions** to intercept and modify user input and model output
- Build **pipe functions** to construct agentic workflows with multiple LLM calls

We now provide you with several different exercises to deepen your understanding and explore different applications. There is most likely insufficient time to do all exercises, hence we recommend you to choose the one that seems most interesting to you. Whereas exercises 1 is more guided, exercise 2 is more open-ended.

1. [Exploring **action functions**](#exercise-1-building-an-action-function)
2. [Using **valves** as user-configurable controls](#exercise-2-adding-valves-to-your-filters)
3. [Extending the agentic coding pipeline with a code patching step.](exercise-3-extending-the-safecoder-agentic-pipeline)

## Exercise 1: Building an Action Function
[Action Functions](https://docs.openwebui.com/features/plugin/functions/action/) will appear as clickable buttons below the generated response from the LLM. Whereas filter functions automatically execute operations on the input prompt or generated response, action functions need to be manually triggered by a user by clicking on the button below the response from the LLM. Several examples of action functions are:

* Summarizing the response from the LLM (_this exercise_)
* [Visualizing a response from a LLM](https://openwebui.com/posts/4319b6c2-5070-43e4-8310-738cd70ae61f)
* [Convert LLM responses in markdown to Word documents](https://openwebui.com/posts/76716d12-5896-4698-a4ac-62fa23dd7251)

In this exercise, we will work on an action function allowing the user to summarize the initial LLM response. Below, you will find a base version of the action function. We will go through setting up an initial version of the summarization function, after which we will give you some suggestions to adapt the function.

{: .action}
> First, we will set up the base version of the action function in your Open WebUI instance. Below this action blok, you can open the base version of the code. You can add it in a similar fashion as before:
> 1. Add the function through the functions tab of the admin panel, save it, and through the `•••` option enable it globally.
> 2. Remaining in the functions tab of the admin panel, and through the valves of the function, change the `Api Base Url` to one of the provided API URLs. Set the `model_name` valve to the corresponding model.
> 3. Now, ask the LLM a question, and click on the `Summarization` icon below the response at the right.

<details markdown="1">
<summary>Show summarization action function code</summary>

```python
from pydantic import BaseModel, Field
from typing import Optional, Literal
import json
import aiohttp
import asyncio

class Action:
    class Valves(BaseModel):
        # LLM Provider Configuration
        llm_provider: Literal["openai", "ollama"] = Field(
            default="openai",
            description="LLM provider to use (openai or ollama)",
        )

        # API Configuration
        api_base_url: str = Field(
            default="https://api.openai.com/v1",
            description="Base URL for the LLM API (e.g., https://api.openai.com/v1, http://localhost:11434/api for Ollama)",
        )
        api_key: str = Field(
            default="",
            description="API key for authentication (not needed for local Ollama)",
        )
        model_name: str = Field(
            default="mistralai/Mistral-7B-Instruct-v0.3",
            description="Model name",
        )

        # Request Configuration
        temperature: float = Field(
            default=0.3,
            description="Temperature for response generation (0.0-1.0) - lower for more focused summaries",
        )
        max_tokens: int = Field(default=500, description="Maximum tokens in response")
        timeout: int = Field(default=30, description="Request timeout in seconds")

    class UserValves(BaseModel):
        show_status: bool = Field(
            default=True, description="Show status messages during processing"
        )
        summary_style: Literal[
            "bullet", "paragraph", "tldr", "executive", "academic"
        ] = Field(
            default="bullet",
            description="Style of summary (bullet points, paragraph, TL;DR, executive, academic)",
        )
        summary_length: Literal["ultra-short", "short", "medium", "detailed"] = Field(
            default="short", description="Length of summary"
        )
        output_format: Literal[
            "styled_section",
            "quote_block",
            "floating_card",
            "separator_line",
        ] = Field(
            default="styled_section",
            description="How to display the summary: styled section, quote block, floating card, or simple separator",
        )
        show_metrics: bool = Field(
            default=True, description="Show compression metrics and statistics"
        )

        ##### HERE YOU CAN ADD MORE USER VALVES #####

    def __init__(self):
        self.valves = self.Valves()

    async def make_llm_request(self, prompt: str, system_prompt: str) -> str:
        """Make a request to the configured LLM provider"""

        headers = {}
        data = {}

        # Configure request based on provider
        if self.valves.llm_provider == "openai":
            headers = {
                "Authorization": f"Bearer {self.valves.api_key}",
                "Content-Type": "application/json",
            }
            data = {
                "model": self.valves.model_name,
                "messages": [
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": prompt},
                ],
                "temperature": self.valves.temperature,
                "max_tokens": self.valves.max_tokens,
            }
            endpoint = f"{self.valves.api_base_url}/chat/completions"

        elif self.valves.llm_provider == "ollama":
            headers = {"Content-Type": "application/json"}
            data = {
                "model": self.valves.model_name,
                "prompt": f"{system_prompt}\n\n{prompt}",
                "temperature": self.valves.temperature,
                "options": {"num_predict": self.valves.max_tokens},
                "stream": False,
            }
            endpoint = f"{self.valves.api_base_url}/generate"

        else:  # custom
            # For custom providers, assume OpenAI-compatible API
            headers = {
                "Authorization": f"Bearer {self.valves.api_key}",
                "Content-Type": "application/json",
            }
            data = {
                "model": self.valves.model_name,
                "messages": [
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": prompt},
                ],
                "temperature": self.valves.temperature,
                "max_tokens": self.valves.max_tokens,
            }
            endpoint = f"{self.valves.api_base_url}/chat/completions"

        # Make the request
        timeout = aiohttp.ClientTimeout(total=self.valves.timeout)
        async with aiohttp.ClientSession(timeout=timeout) as session:
            async with session.post(endpoint, headers=headers, json=data) as response:
                if response.status != 200:
                    error_text = await response.text()
                    raise Exception(f"LLM API error ({response.status}): {error_text}")

                result = await response.json()

                # Extract content based on provider
                if (
                    self.valves.llm_provider == "openai"
                    or self.valves.llm_provider == "custom"
                ):
                    return result["choices"][0]["message"]["content"]

                elif self.valves.llm_provider == "ollama":
                    return result["response"]

        return "Could not generate summary"

    def get_length_instruction(self, length: str) -> str:
        """Get length instruction based on user preference"""
        length_map = {
            "ultra-short": "1-2 sentences maximum",
            "short": "3-5 sentences",
            "medium": "1-2 paragraphs",
            "detailed": "comprehensive with multiple paragraphs",
        }
        return length_map.get(length, "3-5 sentences")

    def get_style_format(self, style: str) -> str:
        """Get format instruction based on style preference"""
        style_formats = {
            "bullet": """Format as bullet points:
• Main point 1
• Main point 2
• Main point 3""",
            "paragraph": "Format as a flowing paragraph with complete sentences.",
            "tldr": "Format as 'TL;DR: [one sentence summary]'",
            "executive": """Format as an executive summary with sections:
**Overview:** [brief context]
**Key Points:** [main findings]
**Recommendations:** [if applicable]""",
            "academic": """Format as an academic abstract with:
**Purpose:** [main objective]
**Findings:** [key results]
**Implications:** [significance]""",
        }
        return style_formats.get(style, style_formats["bullet"])

    async def action(
        self,
        body: dict,
        __user__=None,
        __event_emitter__=None,
        __event_call__=None,
    ) -> Optional[dict]:
        print(f"action:{__name__}")

        # Check if API is configured
        if not self.valves.api_key and self.valves.llm_provider in [
            "openai",
            "anthropic",
        ]:
            if __event_emitter__:
                await __event_emitter__(
                    {
                        "type": "status",
                        "data": {
                            "description": "❌ Please configure API key in settings",
                            "done": True,
                        },
                    }
                )
            return None

        user_valves = (
            __user__.get("valves", self.UserValves()) if __user__ else self.UserValves()
        )

        # Get the message to summarize
        messages = body.get("messages", [])
        if not messages:
            if __event_emitter__:
                await __event_emitter__(
                    {
                        "type": "status",
                        "data": {
                            "description": "No message found to summarize",
                            "done": True,
                        },
                    }
                )
            return None

        # Get the last message or combine multiple messages
        message_to_summarize = messages[-1].get("content", "")

        # Option to summarize entire conversation
        if len(messages) > 1:
            # Check if user wants to summarize the whole conversation
            conversation_context = "\n\n".join(
                [
                    f"{msg.get('role', 'user').upper()}: {msg.get('content', '')}"
                    for msg in messages[-5:]  # Last 5 messages for context
                ]
            )
            if len(conversation_context) > len(message_to_summarize) * 2:
                message_to_summarize = conversation_context

        if not message_to_summarize:
            if __event_emitter__:
                await __event_emitter__(
                    {
                        "type": "status",
                        "data": {"description": "Message is empty", "done": True},
                    }
                )
            return None

        # Show processing status
        if user_valves.show_status and __event_emitter__:
            status_emoji = {
                "bullet": "📋",
                "paragraph": "📄",
                "tldr": "⚡",
                "executive": "💼",
                "academic": "🎓",
            }
            await __event_emitter__(
                {
                    "type": "status",
                    "data": {
                        "description": f"{status_emoji.get(user_valves.summary_style, '📝')} Creating {user_valves.summary_length} summary...",
                        "done": False,
                    },
                }
            )

        # Create the system prompt
        system_prompt = """You are an expert summarizer who creates clear, concise, and accurate summaries. 
        Focus on the most important information and maintain the original meaning."""

        # Create the summary prompt
        length_instruction = self.get_length_instruction(user_valves.summary_length)
        style_format = self.get_style_format(user_valves.summary_style)

        summary_sections = [
            f"📝 **Summary** ({user_valves.summary_style} style, {length_instruction}):"
        ]

        ##### HERE YOU CAN ADD MORE SECTIONS DEPENDING ON A VALVE #####

        summary_prompt = f"""Please summarize the following text.

Requirements:
1. Length: {length_instruction}
2. Style: {style_format}
3. Be accurate and capture the main ideas
4. Maintain objectivity
5. Use clear, concise language

Text to summarize:
"{message_to_summarize}"

Format your response with these sections:
{''.join(summary_sections)}"""

        try:
            # Make the LLM request
            summary = await self.make_llm_request(summary_prompt, system_prompt)

            # Calculate compression ratio
            original_length = len(message_to_summarize.split())
            summary_length = len(summary.split())
            compression_ratio = round((1 - summary_length / original_length) * 100, 1)

            # Format the output based on user preference
            output_content = ""

            if user_valves.output_format == "styled_section":
                # Styled section with clear separation
                output_content = f"""
═══════════════════════════════════════
📝 **SMART SUMMARY** 📝
═══════════════════════════════════════

{summary}

═══════════════════════════════════════
{f'📊 **Metrics:** {compression_ratio}% compression | {user_valves.summary_style} style | {original_length} → {summary_length} words' if user_valves.show_metrics else ''}
"""

            elif user_valves.output_format == "quote_block":
                # Put in a quote block with header
                output_content = f"""
### 📝 Smart Summary ({user_valves.summary_style.title()} Style)

> {summary}

{f'📊 *Compressed by {compression_ratio}% ({original_length} → {summary_length} words)*' if user_valves.show_metrics else ''}
"""

            elif user_valves.output_format == "floating_card":
                # Card style with clear borders and spacing
                output_content = (
                    f"""
╭─────────────────────────────────────╮
│  📝 **SMART SUMMARY** 📝           │
╰─────────────────────────────────────╯

{summary}

╭─────────────────────────────────────╮
│ 📊 {compression_ratio}% COMPRESSION              │
│ 📝 {original_length} → {summary_length} words                │
│ 🎯 {user_valves.summary_style.upper()} STYLE               │
│ 📏 {user_valves.summary_length.upper()} LENGTH             │
╰─────────────────────────────────────╯
"""
                    if user_valves.show_metrics
                    else f"""
╭─────────────────────────────────────╮
│  📝 **SMART SUMMARY** 📝           │
╰─────────────────────────────────────╯

{summary}

╭─────────────────────────────────────╮
│  {user_valves.summary_style.upper()} STYLE - {user_valves.summary_length.upper()} LENGTH    │
╰─────────────────────────────────────╯
"""
                )

            elif user_valves.output_format == "separator_line":
                # Simple separator format
                output_content = f"""
---

**[Smart Summary - {user_valves.summary_style.title()} Style, {user_valves.summary_length.title()} Length]**

{summary}

---
{f'_Compressed by {compression_ratio}% ({original_length} → {summary_length} words)_' if user_valves.show_metrics else ''}
"""

            # Emit the result
            if __event_emitter__:
                # Update status
                if user_valves.show_status:
                    await __event_emitter__(
                        {
                            "type": "status",
                            "data": {
                                "description": f"✨ Summary created! ({compression_ratio}% shorter)",
                                "done": True,
                            },
                        }
                    )

                # Add the summary as a new message
                await __event_emitter__(
                    {
                        "type": "message",
                        "data": {
                            "content": output_content,
                            "role": "assistant",
                            "metadata": {
                                "source": "Smart Summarizer Enhanced",
                                "original_length": original_length,
                                "summary_length": summary_length,
                                "compression_ratio": compression_ratio,
                                "style": user_valves.summary_style,
                                "length": user_valves.summary_length,
                                "output_format": user_valves.output_format,
                                "model_used": self.valves.model_name,
                                "provider": self.valves.llm_provider,
                            },
                        },
                    }
                )

        except Exception as e:
            print(f"Error generating summary: {str(e)}")
            if __event_emitter__:
                await __event_emitter__(
                    {
                        "type": "status",
                        "data": {
                            "description": "❌ Failed to create summary",
                            "done": True,
                        },
                    }
                )

                # Provide error details
                await __event_emitter__(
                    {
                        "type": "citation",
                        "data": {
                            "source": {"name": "Error"},
                            "document": [f"Failed to generate summary: {str(e)}"],
                            "metadata": [
                                {
                                    "source": "Smart Summarizer Action Enhanced",
                                    "provider": self.valves.llm_provider,
                                    "model": self.valves.model_name,
                                    "api_url": self.valves.api_base_url,
                                }
                            ],
                        },
                    }
                )

        return None
```
</details>

Now that you have set up the action function, it is time to explore the functionalities, and gain a more proper understanding of the code behind it.

{: .action}
> Explore the functionalities of the action function:
> * Have a look at the different options in the user valves (summary style, summary length, ...), which you can access by clicking on the `controls` at the top right when in a chat, and then selecting the corresponding valves.
> * When does the summarization work well? When does it not perform well?
> * When looking at the code, can you see where the valves are configured? And where the different options for different configurations are specified?

After you have explored the base functionalities of the summarization function, it is time to give it your own spin. Think about the current functionalities, are there missing features? Are there cases in which the function was not performing up to your standard? 

{: .action}
> Based on your  extend/improve the summarization function. We provide two suggestions, but if you have other ideas to improve the function, feel free to follow those!
> * Add another possible style of the summarization to the options. For example, a humoristic summary or an extremely detailed summary.
> * Add the option to instruct the model to provide key points and/or action items. _Hint: you need to add a user valve, and append an instruction to the prompt, depending on the setting of the valve._

## Exercise 2: Adding Valves to Your Filters

In [Part 1](quickstart_function.md), the PPI filter always redacted detected sensitive information. In practice, you may want to provide users with more control on the behavior of the filter. You can achieve this using **valves**, settings that the user can manage.

### Goal
Extend the PPI filter with one or more **valves**, such as:
- A toggle to enable/disable the redaction of PPI
- A “warn only” mode, in which the user is warned that sensitive information is detected, but it is not redacted.
- A verbosity level for user warnings, which indicates the severity of the warning.

### Reflection
- How do valves turn a function into a reusable *tool* rather than a hard-coded rule?
- Which settings should be exposed and available to users, and which should remain internal?

{: .tip}
> For examples on how to include valves into your function, have a look at some community examples of functions, available [here](https://openwebui.com/?sort=hot). For instance, [this function](https://openwebui.com/posts/65a2ea8f-2a13-4587-9d76-55eea0035cc8) for generating flash cards, or [this function](https://openwebui.com/posts/93efd285-0cd2-4f44-84a1-cf7596bc7bf2) that bypasses model refusals. 

## Exercise 3: Extending the SafeCoder Agentic Pipeline
The third exercise extends the SafeCoder pipeline we worked on in [part 2](agentic.md). In the setting in part 2, we have two LLMs: the first one generates the code, and the second LLM evaluates the code for safety. In this exercise, we add a third stage, in which the initial code is improved based on the insecurities detected before. This approach is a slight simplification of [TypePilot](https://arxiv.org/pdf/2510.11151), a work in which such an agentic framework is used to improve code generation for strongly typed coding languages. 

{: .tip}
> Starting from the code in [part 2](agentic.md), you will have to add two things: a function creating the prompt for the new stage, and an addition of the stage in the `pipe` function. Based on the first two stages in the `pipe` function, you can see how to add the third stage in the function. As before, you can debug by using the `logging` module in Python and subsequently inspecting the logs in the docker container. 


_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
