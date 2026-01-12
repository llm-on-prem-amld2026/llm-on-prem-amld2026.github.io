---
title: "Connecting to your machine"
nav_order: 1
---

Welcome to our workshop. During this workshop, we will work on a dedicated remote compute instance so everyone has the same, clean environment with sufficient CPU/GPU resources, independent of their personal laptop setup. All instances run Ubuntu 24.04, a modern long-term support (LTS) Linux distribution that is widely used in production and provides excellent support for Docker, GPUs, and machine learning tooling.

The first step is to connect to your own compute instance, provided to you by [Exoscale](https://www.exoscale.com/?utm_source=google&utm_medium=cpc&utm_campaign=eu-en-b-exoscale&utm_term=exoscale&gad_source=1&gad_campaignid=12788996154&gbraid=0AAAAACltiW5lUzUBRaS7Au0ZZINQu2kbU&gclid=CjwKCAiA3L_JBhAlEiwAlcWO52vJFNpgN7y3rlQZJUXoVoxIslviqV2--jvcB-LyM_P0iNqR_YCvMRoCy9sQAvD_BwE). Make sure that you have **picked up a paper with an (IP, password) combination from the workshop organizers.** Throughout the set-up, there will be sections where you have to fill in your IP. We indicate this with `<YOURIP>`, which you should then replace with the IP that is on your paper.

## Connecting to the machine through SSH
We will now connect to our machine through SSH. SSH (Secure Shell) is a standard protocol that allows you to securely access and control a remote machine through the command line. In production environments, SSH key-based authentication is strongly recommended, but for simplicity and ease of onboarding during this workshop we will use password-based authentication.

{: .action}
> 1. Open a terminal of your preference: powershell, command prompt, zsh, ...
> 2. Connect to your machine with the command `ssh <YOURIP>@ubuntu`, where you enter the IP that is given to you
> 3. Enter the password, and connect!

<iframe width="560" height="315" src="https://www.youtube.com/embed/m5I15UTNp6Y?si=fbFOFsXy8SV8SOKe" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Installing Docker and Nvidia drivers
Before we can launch our Open WebUI instance, we need to install Docker and Nvidia drivers and toolkit. Docker allows applications to run in isolated, reproducible containers so that software behaves the same on any machine. Nvidia drivers and the Nvidia Container Toolkit enable Docker containers to access the GPU, which is required to run modern AI and machine learning workloads efficiently. If you are unfamiliar with Docker/Nvidia drivers and want to learn more, please refer to the [additional information](../additional_information.md) section. If you simply want to continue to set up your environment, we do so by executing the script below.

{: .action}
Follow the steps below to install Docker and the Nvidia drivers and toolkit. For each step, copy the code and paste and execute it on the command line of your compute instance.

* **Step 1:** Install Docker, by retrieving the official installation script. After installation, the Docker service is restarted to ensure it is ready to use.

```bash
# Install docker
curl -fsSL https://get.docker.com | sh
sudo systemctl restart docker
```

* **Step 2:** Install Nvidia drivers and the Nvidia Container Toolkit. The code snippet first adds Nvidia’s official package signing key, which allows Ubuntu to verify that the software it downloads from Nvidia is authentic. It then adds Nvidia’s package repository to your system so that the required drivers and tools can be installed using Ubuntu’s package manager. After updating the package list, the Nvidia GPU driver, supporting utilities, and the Nvidia Container Toolkit are installed. The toolkit is what enables Docker containers to access the GPU securely and efficiently.
  
```bash
# Install Nvidia drivers needed to run the toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | 
    sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg && 
    curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list |
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' |
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt update
sudo apt install -y nvidia-driver-550-server nvidia-utils-550-server nvidia-container-toolkit
```
* **Step 3:** Reboot the machine. Rebooting is required so the Linux kernel can load the newly installed Nvidia driver. Once the machine restarts, your SSH connection will close; simply reconnect using the same ssh command and password as before.

```bash
sudo reboot
```

{: .tip}
By using the arrow up key, you can quickly use a previous command again. For example, when re-connecting to the machine. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/L_7PrpTrK4s?si=0YjjBL1GIN1Hc2z1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Next step
When you are done with setting up your machine, you can continue to launching your own Open WebUI instance. This is explained in the [next page](openwebui.md).

_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
