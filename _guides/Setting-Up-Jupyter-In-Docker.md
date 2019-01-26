---
layout: guides
title: "Setting up Jupyter inside Docker"
description: "A simple guide to installing, configuring, and starting a Docker containerized Jupyter notebook server."
date: 2019-01-21
---
Docker is a suprisingly useful tool for data science. As the docker environment is essentially frozen, the host platform can be updated and maintained without breaking the solution running inside a Docker container. 

While offers some great tools for productionizing Data Science and Machine Learning solutions, it can be a bit daunting with most of the setup and maintaince needing to be done on the command line.

This guide will run you through the process to get a Docker container running with a Jupyter Notebook Server running inside of it that you can reach from a web browser outside the container.

## Assumptions:
This guide was made and tested on a Mac and an Ubuntu system. As a result, the instructions make certain assumptions about your system.

- You have docker installed on your system.
- You are on mac or linux.

If you do not have Docker installed, you can follow the first half my guide to set up Docker and Nvidia-Docker or head over to the Docker Documentation and follow their instructions.

## Step 1: Starting Up a Docker Container
To get started we are going to spin up a Docker container. We are going to be using the latest version of Ubuntu inside this container. When we start up this container, we will do just a bit of network configuration to allow us to access our Notebook later. Lets get our container running:

```bash
    docker run -ti -p 8888:8888 --name jupyter_container ubuntu:latest bash
```
Lets explore this command real fast. The `-ti` flag stands for 'terminal interactive' it is basically telling docker that we want a terminal when we start up the container. 

The `-p 8888:8888` flag and numbers are telling docker that we want to listen on a particular port (the `-p`). Specifically, we are telling docker to listen to port 8888 on the host machine and that anything that arrives there should be sent to port 8888 in our docker container. The nubers can be (almost) anything you like, however, certain ports are typically reserved for certain service and protocols in networking. So while any number can be used there are a few that may not be the best. To keep it simple, we are using 8888 as that is the default with Jupyter. If you want to use something else keep in mind the `-p` flag has the format `-p <Host Port>:<Container Port>`.

The `--name jupyter_container` option lets us give our docker contianer a name of our choosing. Docker automatically assigns names to containers that are created. We can look at our running containers with `docker ps`:
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
24570b5ff387        ubuntu:latest       "bash"              7 seconds ago       Up 5 seconds                            musing_torvalds
6f34490e9d6d        ubuntu:latest       "bash"              18 seconds ago      Up 16 seconds                           sad_wilson
```
You can see there are two container running with the names 'musing_torvalds' and 'sad_wilson'. These names are given by Docker. The names can be used to restart a container after it has been stopped with `docker restart <name>`. By giving our container a name we know it is easier to restart our container later and pick up where we left off. 

The `ubuntu:latest` section is telling the container we want to our contianer to be running the latest version of Ubuntu. If you wanted a different version you can specify that after the colon. So, if you need Ubuntu 16.04, you would pass `ubuntu:16.04` to the docker command.

Finally, the `bash` option telling Docker that we want bash (a flavor of interactive shell) to be run when the container starts. This is the terminal software that we will interact with while configuring our environment and installing Jupyter.

Running the last command, you have a docker container with the Latest version of Ubuntu running and a terminal open running bash. Now we need to install a few packages inside the container.


## Step 2: Install Packages
By default, the ubuntu:latest container from Docker does not contain a handful of needed packages, including python, pip and nano. If you prefer another terminal based editor instaed of nano, go ahead and and install it instead. Lets grab those packages and get started:

```bash
apt-get update
apt-get install -y python3.7 python3-pip nano
```

Ok, we have python and pip installed in our container, now we need to get jupyter:

```bash
pip install jupyter
```

Great, we now have the bare minimum to get jupyter running in our container. Before we can use it we will need do a bit of configuration.


## Step 3: Configure a Jupyter Server
Jupyter Notebooks can be set up to run like server. We are going to need to configure our jupyter notebook to respond on a specfic port and ip to allow us to access it from outside the container.

The first step is to generate the config file. In my experience Jupyter does not create the config file by default when installing into Docker. To generate our config file execute `jupyter notebook --generate-config`

We now have our config file and it should be located in `/root/.jupyter`.

Next lets set a password. This is not entirely necessary since this is being set up as a local install. However, I still like to do it.

```
jupyter notebook password
Enter password:
Verify password:
```
You will get a confirmation that the password hash was written to your jupyter_notebook_config.json file.

Next, we are going to open our config file and set up our notebook server. To do that we are going to change three settings: the notebook ip, the port the notebook will listen on, and tell the notebook to not automatically open a browser--one does not exist in our container. From `/` run `nano /.jupyter/jupyter_notebook_config.json`. Make sure you are accessing the `.py` file not the `.json`. The `.json` file only has our password hash.

Once we have nano open, hit Control-W to open a search box and seach `.ip` . If you are using something other than nano, utilize that editor's seach function.

Once your are at the right line, uncomment it and inside change the value in the qoutes to be '0.0.0.0'.

Next, use Control-W to search for `.port`, uncomment the line and enter the port number you used when we started up the container. If you are following this guide exactly, use port 8888.

Finally, use Control-W again and seach for `.open_browser`. Uncomment as needed and set the boolean from `True` to `False`. 

That's it! Lets start up the notebook. 

`jupyter notebook --allow-root`

Why the `--allow-root` flag? Jupyter, wisely, does not want you to run the notebook as root. If you have not explored jupyter much you may not know that there is a terminal interface. This terminal has all the privileges that where used when starting the notebook. As a result, if you start the notebook as root, the anyone with access to the web-interface can open a terminal with root access. Generally, this is bad. However, in this case, since we are using Docker and Jupyter locally, this will simplify our interaction with the container since we can just use `pip` and `apt-get` through the notebook webserver now! 

## Step 4: Access from Browser Outside Container
Ok, now that the notebook server is running and configured, lets jump back to our desktop--with the Docker container still running--and open a web browser.Navigate your browser to `localhost:<Port>`, where <Port> is the port number you specified when starting the container. 
 
If everything is running correctly, you should be at the Jupyter password page, type your password in, hit enter, and you have access to your notebook inside Docker!

## Step 5: Commit our changes
If you make changes to a docker container, those changes stay inside that docker container. You can always restart a stopped container and pick up where you left off. If you want to do that you issue the restart command then attach to the container:

```bash
docker restart <name> 
docker attach <name>
```

That will allow you to get back into the same container you were using before. If you want another contianer just like the one you set up here, you would need to recreate it with all the same steps OR commit the changes you made to the base container as a new image that you can use to create new containers. To do that, exit the contianer if it is running by typing `exit` in the terminal of the running contianer, then type:
```bash
docker commit <name> <new_image_name>
```
< name > should be the name you gave the container when you created it or the name of your contianer when you run `docker ps` (note: `docker ps` only shows *running* container, if you need to see stopped containers use the `-a` flag like this: `docker ps -a`).

< new_image_name > should be the name you would like to use for the new image.

So, for me if I named the container `jupyter_server` when I started it, and wanted the new image name to be `jupyter_template`, I would type:
```bash
docker commit jupyter_server jupyter_template
```

Now, if I wanted another Jupyter notebook server, I would just use:
```bash
docker run -ti -p 8888:8888 --name second_server jupyter_template bash
```
*\* NOTE: If your first container is running you will get an error from this command as you are trying to bind the same port on the host. If you want to run two server at the same time, you just need to change the first port number as that it one the container is listening to the host on.*

Commiting containers to images can really save some time in the long run. Once you get a container running with all of your common libraries installed with `pip` and `apt`--or what ever package manager your container uses--I would recomend committing those changes so you do not need to repeat those same steps over and over.

## Docker Hub: Share and Find Containers
Considering the wisdom of an old testament scripture:
    'What has been will be again,
    what has been done will be done again;
    there is nothing new under the sun.'
    - Ecclesiates 1:9
It's unlikely that you need to completely start from scratch when making a container to serve your needs. I, personally, like to do this for simple container so I know what is and is not in my containers. However, there are many containers that have been made to solve common problem or add common functionality and they are shared on docker hub.

Docker Hub, as its name suggest, is similar to GitHub, but just has Docker images that have been shared by the community or Docker itself. Go explore Docker Hub and check out some of the amazing images people have put together to help the community out.