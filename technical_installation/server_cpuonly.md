---
title: "Connecting to Your Machine"
parent: "Technical Installation"
nav_order: 1
---

Welcome to our workshop. During this workshop, we will work on a dedicated remote compute instance so everyone has the same, clean environment, independent of their personal laptop setup. All instances run Ubuntu 24.04, a modern long-term support (LTS) Linux distribution that is widely used in production and provides excellent support for Docker, GPUs, and machine learning tooling. Your instance will not have its own GPU, as GPUs will be shared among participants for efficient use of computational resources.

The first step is to connect to your own compute instance, provided to you by [Exoscale](https://www.exoscale.com/?utm_source=google&utm_medium=cpc&utm_campaign=eu-en-b-exoscale&utm_term=exoscale&gad_source=1&gad_campaignid=12788996154&gbraid=0AAAAACltiW5lUzUBRaS7Au0ZZINQu2kbU&gclid=CjwKCAiA3L_JBhAlEiwAlcWO52vJFNpgN7y3rlQZJUXoVoxIslviqV2--jvcB-LyM_P0iNqR_YCvMRoCy9sQAvD_BwE). Make sure that you have **picked up a paper with an (IP, password) combination from the workshop organizers.** Throughout the set-up, there will be sections where you have to fill in your IP. We indicate this with `<YOURIP>`, which you should then replace with the IP that is on your paper.

## Connecting to the machine through SSH
We will now connect to our machine through SSH. SSH (Secure Shell) is a standard protocol that allows you to securely access and control a remote machine through the command line. In production environments, SSH key-based authentication is strongly recommended, but for simplicity and ease of onboarding during this workshop we will use password-based authentication.

{: .action}
> 1. Open a terminal of your preference: powershell, command prompt, zsh, ...
> 2. Connect to your machine with the command `ssh ubuntu@<YOURIP>`, where you enter the IP that is given to you. Enter _yes_ if it asks you whether you are sure you want to connect.
> 3. Enter the password, and connect!

<iframe width="560" height="315" src="https://www.youtube.com/embed/m5I15UTNp6Y?si=fbFOFsXy8SV8SOKe" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Installing Docker and Nvidia drivers
Before we can launch our Open WebUI instance, we need to install Docker. Docker allows applications to run in isolated, reproducible containers so that software behaves the same on any machine. If you are unfamiliar with Docker and want to learn more, please refer to the [additional information](../additional_information.md) section. If you simply want to continue to set up your environment, we do so by executing the script below.

{: .action}
We will now install docker by retrieving the official installation script. After installation, the Docker service is restarted to ensure it is ready to use. Copy the code below and paste and execute it in your terminal.

```bash
# Install docker
curl -fsSL https://get.docker.com | sh
sudo systemctl restart docker
```

{: .tip}
You can verify whether docker is installed correctly by testing the command `sudo docker ps`, which will list all docker containers on your machine.

<iframe width="560" height="315" src="https://www.youtube.com/embed/qHTGinURvl8?si=cIY98q7sKjpPMwUT" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Next step
When you are done with setting up your machine, you can continue to launching your own Open WebUI instance. This is explained in the [next page](openwebui_cpuonly.md).

_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
