---
title: "Creating a Knowledge Base"
parent: "Powering LLMs with RAG"
nav_order: 1
---

**Retrieval-Augmented Generation (RAG)** is a technique that enhances large language models by giving them access to external knowledge sources at query time. Instead of relying only on what the model learned during training, a RAG system retrieves relevant documents from a knowledge base and uses them as context when generating an answer. This makes responses more accurate, up to date, and grounded in specific data.

In practice, RAG allows you to connect LLMs to your own documents—such as reports, manuals, or databases—so they can answer questions based on information you control. In this exercise, you will build a simple RAG system step by step and explore how retrieval settings influence what your model can (and cannot) do.

## Setting up your first RAG system
We will now set up our first RAG system, using a report on the state of AI from McKinsey as the underlying data source. First, we will create our knowledge base. Such a knowledge base can contain manually written text documents, PDFs, Word documents, or web pages. With knowledge bases, you can structure your documents in different categories, and you can specify to which datasets your model should have access. 

{: .action}
> 1. Download the McKinsey report on the state of AI <a href="../assets/McKinsey_AI_2025.pdf" download>here</a>.
> 2. On the left side of your chat interface, select the bottom option **Workspace**
> 3. At the top of the screen, select **Knowledge**, and select the option **+ New Knowledge** at the top. Fill in the fields, giving the knowledge base a name and description. Do not forget to set the knowledge base to **public**.
> 4. Now, add content to the knowledge base, and select the PDF file you downloaded in step 1.

Next, we will create a custom model that has our knowledge base linked to it. Follow the steps below to do so:

{: .action}
> 1. From within the Workspace section, click on **Models** at the top, and click on **+ New Model**
> 2. Give your new model a name, a description, and select the base model (either Qwen or Mistral)
> 3. Next, click on **Select Knowledge**, and add your knowledge base. Don't forget to **Save & Create** your model at the end.

{: .note}
> When composing your new model, have a look around at the options you have. For example, you could add a system prompt here and select which capabilities your model has.

Now you have composed your own RAG system! It is time to explore how it performs in this base setting, what can it do and what can it not do? Based on these observations, you will try to improve upon the base setting in the next part. 

{: .action}
> Play around with your new RAG model. Have a look at the McKinsey report yourself, and ask questions to your LLM based on the report. What data is it able to retrieve and when does it fail? Does it do better on generic or specific questions? 

<iframe width="560" height="315" src="https://www.youtube.com/embed/_-5nSR31CP8?si=-95RUPwDPJto8UOH" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Modifying the Retrieval Settings
It is now time to explore all the different settings for RAG systems that Open WebUI offers. Below, you can find an overview of the main settings that we can adjust.

| Setting | Effect | Recommendation |
| :--- | :--- | :--- |
| **Top K** | Limits the number of retrieved documents or chunks to the top K. | **Set to 5** (Good balance of speed vs context) |
| **Full Context Mode** | Feeds the *entire* document to the LLM. | **KEEP OFF** (See warning below) |
| **Hybrid Search** | Combines vector search with keyword-based search (BM25). | **Optional** (Requires extra downloads) |
| **RAG Template** | Defines the prompt template for retrieval-augmented generation. | Safe to experiment |
| **System Prompt** | Sets the base instructions or behavior for the model. | Safe to experiment |

{: .warning}
> **⚠️ WARNING: Do NOT enable "Full Context Mode"**
> The McKinsey report is over 50 pages long. If you enable "Full Context Mode," the system will try to force the entire PDF into the LLM's short-term memory. This will likely cause **local models to crash (Out of Memory error)** or freeze your interface. 
> 
> **Please ensure:**
> * **Full Context Mode:** OFF
> * **Top K:** 5 (or similar low number)

{: .action}
> Based on your findings in the last step, change your RAG system and inspect how the performance changes. If you are not sure where to start, below we provide several options:
> * Try changing **Top K** from 5 to 2. Does the model start missing details? Try raising it to 10. Does the model get confused or slow down?
> * Look at the _RAG Template_, what do you like about it, what don't you like? What would you change?
> * (Optional) Have a look at Hybrid Search (for more information, read [this article](https://medium.com/@csakash03/hybrid-search-is-a-method-to-optimize-rag-implementation-98d9d0911341)), does it improve the retrieved information?
> * If you generally want different behavior of your LLM, try to adjust the RAG template or system prompt

## Next Step
You have now succeeded at setting up your first RAG system, well done! We will now move onto a second exercise, where we will allow our LLM to answer questions regarding a SQL database. Go to the [next exercise](sql.md).

_Authors: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799), [Elena Nazarenko](https://www.linkedin.com/in/lena-nazarenko/)_
