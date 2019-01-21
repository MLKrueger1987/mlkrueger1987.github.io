---
layout: guides
title: "Using Jupyter inside Docker"
date: 2019-01-21
---
# Setting Up Jupyter Notebook inside a Docker Container


## Step 1: Starting Up a Docker Container
- Expose Port
- Mount Directory
- Start Bash


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

Finally, use Control-W again and seach for `.open_browser`. Uncomment is needed and set the boolean from `True` to `False`. 

That's it! Lets start up the notebook. 

`jupyter notebook --allow-root`

Why the `--allow-root` flag? Jupyter, wisely, does not want you to run the notebook as root. If you have not explored jupyter much you may not know that there is a terminal interface. This terminal has all the privileges that where used when starting the notebook. As a result, if you start the notebook as root, the anyone with access to the web-interface can open a terminal with root access. Generally, this is bad. However, in this case, since we are using Docker and Jupyter locally, this will simplify our interaction with the container since we can just use `pip` and `apt-get` through the notebook webserver now! 

## Step 4: Access from Browser Outside Container
Ok, now that the notebook server is running and configured, lets jump back to our desktop--with the Docker container still running--and open a web browser.Navigate your browser to `localhost:<Port>`, where <Port> is the port number you specified when starting the container. 
 
If everything is running correctly, you should be at the Jupyter password page, type your password in, hit enter, and you have access to your notebook inside Docker!

## Step 5: Commit our changes


## Docker Hub: Share and Find Containers