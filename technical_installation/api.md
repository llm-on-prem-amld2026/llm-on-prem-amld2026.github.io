---
title: "Adding an Inference Endpoint"
parent: "Technical Installation"
nav: 3
---
## Adding an Inference Endpoint 
In this part of the workshop, we introduce the concept of **API endpoints** and show how they are used to connect external model providers to Open WebUI. We will also configure one endpoint together, so that you can see how a new model becomes available in the interface.

### Understanding API endpoints
An API (Application Programming Interface) allows different software systems to communicate with each other. An API endpoint is a specific URL that a program can send requests to in order to send a prompt, receive the model's response and manage authentication and permissions. In our case, Open WebUI can connect to multiple LLM providers via API endpoints. Currently, only our local Ollama is configured, in this section we will add an external API endpoint. Each endpoint defines:

* Where the model server is running (the URL)
* How to authenticate (usually via an API key or token)
* Which models become available in the UI

API endpoints are generally configured to serve multiple users simultaneously. The figure below displays what such a set-up looks like. Multiple API clients send their requests to a central load balancer, which distributes the incoming traffic across several API gateways to ensure scalability and high availability. Each API gateway then routes the requests to the appropriate backend API services, handling concerns such as authentication, rate limiting, and request routing. Because of this system, each client only interacts with a unified entry point while the system efficiently balances load and manages access to multiple underlying services.

![](../assets/add_api.png)

### Adding an API endpoint in Open WebUI
We will now add two new OpenAI-compatible endpoint to demonstrate how this works. We provide an endpoint for two models: `Qwen/Qwen2.5-7B-Instruct` and `mistralai/Mistral-7B-Instruct-v0.3`, with the following URLs:

| Model                              | API URL                                                                 |
|------------------------------------|-------------------------------------------------------------------------|
| Qwen/Qwen2.5-7B-Instruct           | https://b2db0789-5f8e-47da-892d-40827df4a57a.inference.at-vie-2.exoscale-cloud.com/v1 |
| mistralai/Mistral-7B-Instruct-v0.3 | https://3aa48518-6930-4630-9f64-05b5e6cea69f.inference.at-vie-2.exoscale-cloud.com/v1 |

The API keys (secret) are available in an encrypted zip file in [this Google Drive](https://drive.google.com/drive/folders/1puP7SCZqm_W_MULqPkgFAyiVGQyVniTc?usp=sharing), the password to the zip file should now be visible on the screen. The steps below will now guide you through adding the API endpoint to your Open WebUI instance.

{: .action}
> 1. Go to the Admin Panel in Open WebUI, and navigate to Settings -> Connections
> 2. For each of the endpoints, under OpenAI API, add a new connection (plus icon)
> 3. Enter:
>      * The provided URL in the endpoint field
>      * The provided API key in te Bearer token field
> 4. Save the configuration. 

### Verifying the connection
You can now return to the main chat interface, and start a new chat. When opening the model selection dropdown, you should now see both the Qwen and Mistral model available in the list. You can select it and start a conversation, to see whether the API endpoints work.

### What's next
Now that you have set up your Open WebUI instance, we can start exploring all different functionalities, take some time to browse around! The next part of the workshop will be [Open WebUI Functions](../functions/quickstart.md).

_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
