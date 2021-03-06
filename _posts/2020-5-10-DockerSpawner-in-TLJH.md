---
layout: post
title: "Using DockerSpawner with The Littlest JupyterHub"
---

[The Littlest JupyterHub](https://tljh.jupyter.org) (TLJH) is a simple distribution of JupyterHub - that is, a nice easy standardised 
way to run your own JupyterHub server on a single machine, with a lot of the config taken care of for you.

One thing currently missing as standard is the option to use Docker for your individual users' Jupyter servers. Work continues to 
automate this, but for now here are the instructions for getting it running.

## Install Docker

You need the Docker daemon to be installed and running on the server. When you installed TLJH, you should have seen some instructions 
for connecting to your server by SSH. Do that, and enter these four command lines:

{% highlight bash %}
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update && sudo apt install -y docker-ce
{% endhighlight %}

If you want to understand these commands in more detail, please see [this guide](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04).

## Install DockerSpawner

Next, we need to install a component called DockerSpawner (used by JupyterHub to start individual users' Jupyter servers within Docker containers).

{% highlight bash %}
sudo /opt/tljh/hub/bin/python3 -m pip install dockerspawner jupyter_client
{% endhighlight %}

## Tell TLJH to use DockerSpawner

Then add some config to TLJH telling it to use DockerSpawner.

If you are comfortable using the vi text editor, create this new config file:

{% highlight bash %}
sudo vi /opt/tljh/config/jupyterhub_config.d/dockerspawner_tljh_config.py
{% endhighlight %}

and paste the following contents:

{% highlight python %}
c.JupyterHub.spawner_class = 'dockerspawner.DockerSpawner'
 
c.DockerSpawner.image = 'ideonate/jh-voila-oauth-scipy:latest'
 
from jupyter_client.localinterfaces import public_ips
 
c.JupyterHub.hub_ip = public_ips()[0]
 
c.DockerSpawner.name_template = "{prefix}-{username}-{servername}"
{% endhighlight %}

Alternatively, to avoid using vi, just run this command instead:
{% highlight bash %}
sudo curl https://gist.githubusercontent.com/danlester/4a0ad6c52719679142a290d303faa3db/raw/d59707f95cba0399ff726392569545cbd46bda33/dockerspawner_tljh_config.py > /opt/tljh/config/jupyterhub_config.d/dockerspawner_tljh_config.py
{% endhighlight %}

## Download the Docker image

To avoid waiting the first time you start a Jupyter server, ideally you should 'pull' the server image that we are using, just 
so it's ready in advance:

{% highlight bash %}
sudo docker pull ideonate/jh-voila-oauth-scipy:latest
{% endhighlight %}

## Reload TLJH with the new config

To restart TLJH, run:

{% highlight bash %}
sudo tljh-config reload
{% endhighlight %}

And that's it! Hopefully this will become even simpler if a plugin is developed to do all of this in one simple tljh-config command.

Please [get in touch](/contact) if you have any questions about this.


