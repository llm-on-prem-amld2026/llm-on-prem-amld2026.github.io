---
title: "Connecting to your machine"
parent: "Technical installation"
nav_order: 1
---

Welcome to our workshop. The first step is to connect to your own compute instance, provided to you by [Exoscale](https://www.exoscale.com/?utm_source=google&utm_medium=cpc&utm_campaign=eu-en-b-exoscale&utm_term=exoscale&gad_source=1&gad_campaignid=12788996154&gbraid=0AAAAACltiW5lUzUBRaS7Au0ZZINQu2kbU&gclid=CjwKCAiA3L_JBhAlEiwAlcWO52vJFNpgN7y3rlQZJUXoVoxIslviqV2--jvcB-LyM_P0iNqR_YCvMRoCy9sQAvD_BwE). Make sure that you have **picked up a paper with an (IP, password) combination from the workshop organizers.** 

## Connecting to the machine through SSH
We will now connect to our machine through SSH

{: .action}
> 1. Open a terminal of your preference: powershell, command prompt, zsh, ...
> 2. Connect to your machine with the command `ssh YOURIP@ubuntu`, where you enter the IP that is given to you
> 3. Enter the password, and connect!

<iframe width="560" height="315" src="https://www.youtube.com/embed/m5I15UTNp6Y?si=fbFOFsXy8SV8SOKe" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Installing Docker and Nvidia drivers
Before we can launch our Open WebUI instance, we need to install Docker and Nvidia drivers and toolkit. If you are unfamiliar with Docker/Nvidia drivers and want to learn more, please refer to the [additional information](../additional_information.md) section. If you simply want to continue to set up your environment, we do so by executing the script below.

{: .action}
Copy the script below on the command line of your compute instance, and execute it.

```bash
# Install docker
curl -fsSL https://get.docker.com | sh
sudo systemctl restart docker

# Install Nvidia drivers needed to run the toolkit
# add nvidia production package repository
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | 
    sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg && 
    curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list |
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' |
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
# install nvidia drivers
sudo apt update

sudo apt install -y nvidia-driver-550-server nvidia-utils-550-server nvidia-container-toolkit
sudo reboot
```
Running the script will have made you exit the machine, as it needs to reboot. Simply connect again by entering your password.

{: .tip}
By using the arrow up key, you can quickly use a previous command again. For example, when re-connecting to the machine. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/L_7PrpTrK4s?si=0YjjBL1GIN1Hc2z1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
