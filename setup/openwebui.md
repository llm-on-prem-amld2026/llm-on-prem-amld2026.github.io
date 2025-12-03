---
title: "Setting up your own Open WebUI"
parent: "Technical installation"
nav_order: 2
---

Now that you have set up your machine, we can continue to launching our Open WebUI instance. 

## Launching our Open WebUI instance
We will set up an [Open WebUI](https://openwebui.com/) instance, with LLMs running through [Ollama](https://ollama.com/). We will run both services through Docker, which leads to a **robust** and **replicable** architecture. If you want to learn more about Open WebUI, Ollama and Docker, we provide additional information in this Section. For now, we recommend you to continue with the set-up. We will need the following commands:

| Command        | Function          | 
|:-------------|:------------------|
| mkdir           | make a directory |
| nano | create / edit a file   |
| cd           | go into or out of a directory      |

We can now set up and launch our Open WebUI instance.

{: .action}
1. Create the directory for openwebui: `mkdir openwebui`
2. Move into the openwebui directory: `cd openwebui`
3. Create the docker compose file: `nano docker-compose.yaml`, and paste the code below. You can exit and save the file through the commands `ctrl X + Y`
4. Now you can start both Ollama and Open WebUI by running `sudo docker compose up -d`
5. Verify with `sudo docker ps` that both containers run well

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
Here we describe the principal configuration options for Open WebUI that are set in the docker compose file.
* _volumes_: here we specify where the data will be stored, in our case in a folder `data_openwebui`
* _depends_on_: this makes sure that Ollama starts before Open WebUI starts
* _ports_: we choose to map the port 8080 in the container to port 3000 on the machine. Therefore, our Open WebUI will be available on port 3000.
* _environment_: here we specify where Ollama is running. We could also secure our application more tightly by specifying a secret key.


## Accessing the Open WebUI instance
Now we can access our Open WebUI instance at the following address: `http://YOURIP/3000`. Here, you can set up your login credentials, and start your first chat.

{: .action}
1. Access your Open WebUI instance, and set up your login details.
2. 

## Video
Here will be a demonstrative video.
