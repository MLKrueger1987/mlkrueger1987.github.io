---
layout: guides
title: "Setting up Docker and Nvidia Docker on Ubuntu"
date: 2019-01-21
---
# Setting up Nvidia-Docker and Docker on Ubuntu
### The easiest way to get your Nvidia GPU up and running for Machine Learning 

Its no secret, utilizing a GPU for training models can save a lot of time. It's
not a silver bullet for long training times for deep models it can decrease 
training times significantly. Unfortunately, getting your GPU up and running to
use for machine learning can be a giant head ache.

The purpose of this guide is help you get your environment set up in the easiest
way I have found so far. To do this, we are going to set up Docker and 
Nvidia-Docker to allow the use of Docker container from Nvidia GPU cloud.

## Docker? 
Some of you may be wondering what Docker is. Docker is containerization platform.
At a very high level, its a virtualization platform. However, instaed of running
a complete oppertating system, like a virtual machine ("VM"), a container only 
runs a operating system kernel making it much lighter weight than a VM. Each 
container is isolated from the others which allows each container to use a 
different kernel and  different libraries. This is a pretty simplistic explanation
of what Docker is, but it should suffice for our purposes.

So now with an idea of what docker is, lets get into why we want to use it.

## Why Docker?
Getting everything configured to use your GPU for ML can be a real pain. First,
you obviously need to have the correct drivers. Then, you need to have the right
libraries installed. This is not terrible to get lined up. However, at somepoint
you will update your system, a driver will change, or a library will be update,
or maybe even a dependency of a library will change. So what? Well, often times
these seemingly innocent updates will break your system. 

## The Guide:
I am using Ubuntu 18.04 LTS while setting up this guide. The process is fairly general and you can likely adapt it to recent version of Ubuntu or its various flavors with out much difficulty. However, as I am using Bionic Beaver, I am only directly addressing the install challenges for that distro.

### Assumptions:
- You are running Ubuntu 18.04 LTS.
- You are comfortable using the terminal and `apt`.
- You have sufficient privileges to install packages
or otherwise modify your system.
- You do not have docker installed.
- You are using a computer with x86_64 or amd64 architechure. If you are unsure, you probably are ok as these are the most common architechtures for consumer hardware.

### Step 1: Nvidia Drivers

### Step 2: Get Docker

#### Installation
We are going to install Docker Community Edition (Docker CE). These are the same instructions you can find in the the Docker docs. If you run into problems installing docker checkout their official docs [here](https://docs.docker.com/install/linux/docker-ce/ubuntu "Docker Docs"). 

There are a few ways to install docker-ce. We are going to use repo method in order to facilitate easier upgrading later. First step is to give `apt` the avility to use a repo over https. 
```bash
sudo apt-get install \
 apt-transport-https \
 ca-certificates \
 curl\
 gnupg2\
 software-properties-common
```

Next, we need to add Docker's official GPG key:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Now verify that you have the key with the correct finger print:
```
9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88
```

We can verify by checking for the last 8 characters of the above finger print:

```bash
sudo apt-key fingerprint 0EBFCD88
```

You should get output that is similar to the below:
```
pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid   Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
```
Ok, now we have the packages and the GPG key we need. Now we can add the repo and install:

```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Now, we need to update `apt` to have the packages from the just added repo.

```bash
sudo apt-get update
```

Finaly, we can install the latest version:

```bash
sudo apt-get install docker-ce
```

Docker comes with a neat little 'hello-world' container so we can check that everything works correctly. Personally, I always need to start the docker daemon the first time. So lets do that then start the hello-world container.


```bash
sudo systemctl start docker
sudo docker run hello-world
```
Since you are running docker for the first time, you should see container being fetched followed by the following output:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
Congratulations, you have docker installed and running! Next, we are going to configure our system to make using docker more convenient.

#### Configuration

Docker requires a daemon to be running but oddly, by default, does not set that daemon to run automatically at boot. We can easily tell Ubuntu that we want this daemon started automatically. This is not a required addition but does add some convenience. To start the daemon automatically, simply enter the followin in the terminal:
```bash
sudo systemctl enable docker
```
Now, the Docker Daemon should start automatically when you boot up.

Docker, by default, requires elevated privilleges to run container. This means, you always need to use ```sudo``` when interacting with docker. Personally, I don't mind this much, but it can be a bit tedious so I am going to show you how to user docker without needing to use sudo everytime.

Docker typically creates a 'docker' group when it is installed. However, it does not add any users to the group.
So we are going to add our user to the group. Just to be sure we have the group, we will try to create it first:
```bash
sudo groupadd docker
```
If you get a message saying the group already exists, don't worry.

Now to add our user to the group:

```bash
sudo usermod -aG docker $USER
```
Reboot to allow your user to have their group membership re-evaluated.

Now your user is added to the docker group and you should be able run docker commands without needing to `sudo` everytime.


### Step 3: Get Nvidia Docker 
Now that we have Docker installed, we need to install Nvidia-Docker. Docker does not leverage GPU's by default. In order to make use of your GPU for compute power, Nvidia kindly developed NVIDIA-Docker to allow for GPU leverage and portability between systems.

There are only two prerequists for using Nvidia-Docker:
- A Docker installation
- An up-to-date, relatively, Nvidia driver.

If you have been following this guide, you should have both requirements satisfied.

#### Installation
The installation for Nvidia Docker is very similar to that of Docker. We are going to add the repositories for `nvidia-docker`, add the GPG key for nvidia, install `nvidia-docker`, and finally test that the install was successful.

To start lets get the Nvidia GPG key:
```bash
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
```
Now we can add the correct repo:
```bash
curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu18.04/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```
As with the Docker install, we need to update the list of packages for apt so it can see the new repo:
```bash
sudo apt-get update
```
Now we can install `nvidia-docker2`:
```bash
sudo apt-get install nvidia-docker2
```
Now we want to test that everything was installed correctly and your install can acess your GPU. Lets reload the Docker Daemon and test nvidia-docker:
```bash
sudo pkill -SIGHUP dockerd
docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
```
If everything is intalled correctly, you should see the output of nvidia-smi which should display the name of your GPU and other informtion. It should look something like this:
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 390.77                 Driver Version: 390.77                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  TITAN V             Off  | 00000000:41:00.0  On |                  N/A |
| 28%   35C    P8    27W / 250W |    487MiB / 12065MiB |      1%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```
Great! You now have docker and nvidia-docker installed.

## Part 2:
In the second part of the guide we will setup Jupyter notebook inside a docker container and allow you to interact with your notebook inside Docker from your web-browser.


