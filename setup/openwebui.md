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

Please follow the following steps:

* Create the directory for openwebui: `mkdir openwebui`
* Move into the openwebui directory: `cd openwebui`
* Create the docker compose file: `nano docker-compose.yaml`

Now paste the following code in the docker compose file:

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

├── openwebui
│   ├── docker-compose.yaml
│   └── docker-compose.yaml.save

{: .tip}
> Test whether this works
> Another note

## Accessing the Open WebUI instance

