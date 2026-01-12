---
title: "Setting up your own Open WebUI"
nav_order: 2
---

Now that you have set up your machine, we can continue to launching our Open WebUI instance. 

## Launching our Open WebUI instance
We will set up an [Open WebUI](https://openwebui.com/) instance, with LLMs running through [Ollama](https://ollama.com/). Ollama is a lightweight service that downloads, manages, and runs large language models, while Open WebUI provides a browser-based interface for interacting with those models through a chat-style UI. Together, they separate model execution from user interaction, making it easy to run and manage LLMs on a remote machine. We will run both services through Docker, which leads to a **robust** and **replicable** architecture. If you want to learn more about Open WebUI and Ollama and Docker, we provide additional information in [this section](<Additional Information/openwebui.md>). For now, we recommend you to continue with the set-up. We will need the following commands:

| Command        | Function          | 
|:-------------|:------------------|
| mkdir           | make a directory |
| nano | command-line text editor   |
| cd           | go into or out of a directory      |

We can now set up and launch our Open WebUI instance.

{: .action}
> 1. Create the directory for openwebui: `mkdir openwebui`
> 2. Move into the openwebui directory: `cd openwebui`
> 3. Create the docker compose file: `nano docker-compose.yaml`, and paste the code below. You can then save the file with `ctrl O + enter`, and exit nano with `ctrl X`. These steps are displayed in the image below.

![](assets/images/nano.jpg)

{: .action}
Now that we have set up the docker compose file, we are ready to launch our Open WebUI instance. The only thing you have to do is enter the command `sudo docker compose up -d`, which spins up the container in the background. Make sure you are in the openwebui folder when executing the command. After the containers have started, you can verify that they are running with `sudo docker ps`. You can inspect the logs with `sudo docker logs ollama` and `sudo docker logs open-webui`.

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    ports:
      - 11434:11434
    volumes:
      - /home/ubuntu/ollama:/root/.ollama
    container_name: ollama
    deploy:
      resources:
        reservations:
          devices:
          - driver: nvidia
            capabilities: ["gpu"]
            count: 1  # Adjust count for the number of GPUs you want to use
    tty: true
    restart: unless-stopped

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    volumes:
      - /home/ubuntu/data_openwebui:/app/backend/data
    depends_on:
      - ollama
    ports:
      - 3000:8080
    environment:
      - 'OLLAMA_BASE_URL=http://ollama:11434'
      - 'WEBUI_SECRET_KEY='
      - 'GLOBAL_LOG_LEVEL=DEBUG'
    extra_hosts:
      - host.docker.internal:host-gateway
    restart: unless-stopped
```

{: .note}
> Here we describe the principal configuration options for Open WebUI that are set in the docker compose file.
> * _volumes_: here we specify where the data will be stored, in our case in a folder `data_openwebui`
> * _depends_on_: this makes sure that Ollama starts before Open WebUI starts
> * _ports_: we choose to map the port 8080 in the container to port 3000 on the machine. Therefore, our Open WebUI will be available on port 3000.
> * _environment_: here we specify where Ollama is running. We could also secure our application more tightly by specifying a secret key.

<iframe width="560" height="315" src="https://www.youtube.com/embed/fTbv9woR-F8?si=-7UyDPDZJy4kR06b" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Accessing the Open WebUI instance
Now we can access our Open WebUI instance at the following address: `http://<YOURIP>/3000`. Here, you can set up your login credentials, and start your first chat.

{: .action}
> 1. Access your Open WebUI instance at `http://<YOURIP>:3000/`, and set up your login details. 
> 2. Load a model from [Ollama](https://ollama.com/search), by clicking on the model selection button, entering the name, and choosing _Pull from Ollama.com_ We recommend for a short test the model `llama3.1:8b`.
> Congrats! You have now set up your very own LLM interface! Take some time to look around and explore the possibilities.

{: .warning}
You are limited to XXXX GB of memory (RAM). Make sure to not load too many models simultaneously, otherwise you may overload this memory. If you want to remove a model, you can do so by going to the admin panel > settings > connections. Here, next to `http://ollama:11434`, you will see a "manage" button, where you can delete models. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/smPmNy_bcGg?si=LOxC-vHxRPKI_IwA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Next step
Now that you have set up your Open WebUI instance, we can start exploring all different functionalities, take some time to browse around! The next part of the workshop will be [RAG](RAG/exercise.md).

_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
