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

_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
