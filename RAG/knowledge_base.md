---
title: "Creating a Knowledge Base"
parent: "Powering LLMs with RAG"
nav_order: 1
---


TO DO: SHORT INTRO TO RAG

## Setting up your first RAG system
We will now set up our first RAG system, using a report on the state of AI from McKinsey as the underlying data source. First, we will create our knowledge base. Such a knowledge base can contain manually written text documents, PDFs, Word documents, or web pages. With knowledge bases, you can structure your documents in different categories, and you can specify to which datasets your model should have access. 

{: .action}
> 1. Download the McKinsey report on the state of AI <a href="assets/McKinsey_AI_2025.pdf" download>here</a>.
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

## Modifying the Retrieval Settings
Based on your findings in the 

## Next Step
