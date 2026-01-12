---
parent: "Additional Information"
title: "Docker, Cuda and Nvidia drivers"
nav_order: 1
---

Here you will find additional information on several parts of the workshop.

## Docker, Cuda and Nvidia drivers

**Docker** is a platform that allows you to package applications and all their dependencies into lightweight, portable units called containers. There are two critical advantages of using this setup. First, it is replicable across different machines. Whereas using conda will depend on the operating system and the specific system libraries installed, Docker containers behave the same way regardless of the host environment. Second, Docker helps isolate the application from the rest of your system. This reduces the risk of version conflicts, makes cleanup easier, and allows you to experiment without affecting other projects or software installed on your machine. For more information, we refer you to [this beginner tutorial on Docker.](https://docker-curriculum.com/)

**CUDA** is NVIDIA’s computing platform that allows software to run calculations on the GPU instead of the CPU. GPUs are particularly well-suited for the kind of large matrix operations used by modern machine learning models, which makes CUDA a key component for running LLMs efficiently. In practice, you usually do not interact with CUDA directly. Instead, the LLM software is written to use CUDA behind the scenes, as long as the right CUDA libraries are installed in your Docker container. As long as your system meets the requirements, the container can take advantage of GPU acceleration without requiring you to manually manage CUDA versions on your host machine.

**NVIDIA drivers** and the **NVIDIA Container Toolkit** are what make GPU usage possible from within Docker. The NVIDIA driver is installed on your host operating system and acts as the low-level interface between software and the GPU hardware. Docker containers cannot access the GPU by default, so the NVIDIA Container Toolkit provides the necessary integration layer that allows containers to see and use the GPU through the host driver. This separation is important: the driver lives on the host, while CUDA and the machine learning libraries live inside the container. When everything is set up correctly, this allows Dockerized applications like Open WebUI and LLM backends to use the GPU seamlessly, while keeping the system modular and easier to maintain.

_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
