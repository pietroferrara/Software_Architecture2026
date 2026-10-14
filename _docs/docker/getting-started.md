---
title: 2. Getting Started
category: 01. Docker
order: 2
---
<h2>Contents</h2>
* toc
{:toc}
In this section, we will see step-by-step how we can create images and initialize containers. To follow properly, you should have <a target="_blank" rel="noopener noreferrer" href="../introduction">Docker</a> (Docker Desktop or the Docker Engine) installed properly on your machine.

## The Docker Hub
We want now to play around with Docker to understand better how it works. We will use the Docker CLI (i.e., the docker command from a terminal): it works the same way on every operating system. Everything we do can also be done from Docker Desktop, and we will see it at the end of this section. The easier way to see Docker in action is to download pre-defined images from the Docker Hub.  

The <a target="_blank" rel="noopener noreferrer" href="https://hub.docker.com/">Docker Hub</a> is a container image registry. From Docker Hub, you can fetch ready-to-use images from which you can base your application. Do you want to deploy in a container an application written in Python? You can download a <a target="_blank" rel="noopener noreferrer" href="https://hub.docker.com/_/python">Docker image</a> that has python already installed and configured.

## Image tags
A docker image has a tag. This permits you to specify which version of the Docker image you want to use. For example, suppose that your python application relies on libraries that were developed with python 3.13. Probably you don't want to use the latest version of python, since the library could not work properly on newer versions. So, you should use a python 3.13 Docker image (e.g., python:3.13). If you don't specify a tag, Docker uses the **latest** tag: this is fine for experiments, but in a real project you should always specify a tag, otherwise your application may suddenly break when a new version of the image is released. You can see the available tags of an image directly from the image page of the Docker Hub:
![Docker container]({{ site.baseurl }}/images/docker_hub_tags.png)


## Getting Started
Open a terminal and type the following command:
{% highlight bash %}
~/Projects/sw-arch/docker docker run -d -p 8080:80 --name tutorial docker/getting-started
Unable to find image 'docker/getting-started:latest' locally
latest: Pulling from docker/getting-started
...
Status: Downloaded newer image for docker/getting-started:latest
481bdac80071b4acf401dfe5e273a938d47d356f83ae15250859494803ee9574
{% endhighlight %}
This command downloads from the Docker Hub an image (docker/getting-started) that contains a simple tutorial, and then it creates and runs a container from that image. We will see the docker command in detail in the next section; for now:
1. **-d** runs the container in the background (detached mode);
2. **-p 8080:80** maps port 8080 of our machine (the host) to port 80 of the container;
3. **--name tutorial** gives a name to the container (the latter is facultative: if you don't specify it, Docker chooses a random name for you).

The download requires a bit. Then, you can check that the container is running with the docker ps command:
{% highlight bash %}
~/Projects/sw-arch/docker docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS         PORTS                                   NAMES
481bdac80071   docker/getting-started   "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   tutorial
{% endhighlight %}
To see the deployed application in action, go on <a target="_blank" rel="noopener noreferrer" href="http://localhost:8080">localhost:8080</a> (or, if you choose a different host port, substitute the port on the URL). If you see something like this, it means that you have launched your first container correctly :) :
![Docker Getting Started 5  ]({{ site.baseurl }}/images/docker_gettingstarted_5.png)
We will turn back on this later. Now it is time to see how we can interact with the container.

You can stop, restart, and delete a container using its name (or its ID):
{% highlight bash %}
~/Projects/sw-arch/docker docker stop tutorial
tutorial
~/Projects/sw-arch/docker docker start tutorial
tutorial
{% endhighlight %}
You can see the container logs with docker logs, and you can open a terminal inside the container with docker exec:
{% highlight bash %}
~/Projects/sw-arch/docker docker logs tutorial
...
~/Projects/sw-arch/docker docker exec -it tutorial sh
/ # cat /etc/alpine-release
3.17.0
/ # exit
{% endhighlight %}
The flags **-it** connect your terminal to the container, so that you can type commands inside it. As you can see, this image is based on Alpine, a very small Linux distribution.  
Finally, to delete the container (the -f flag stops it first, if it is running):
{% highlight bash %}
~/Projects/sw-arch/docker docker rm -f tutorial
tutorial
{% endhighlight %}

### Docker Desktop
If you installed Docker Desktop, you can do the same operations from its interface. From the Containers tab, you can stop, pause, restart, and delete containers. If you click on the container name, you can find a simple interface that permits you to see the container logs and some statistics (like, for example, CPU and memory usage), access the container's files, and interact with the container using a terminal (the interface may look slightly different depending on your version):
![Docker Getting Started 6]({{ site.baseurl }}/images/docker_gettingstarted_6.png)

## Exercises
1. Try to **install python3** inside the container created on the Getting Started section (open a terminal inside it with ```docker exec -it tutorial sh```, then run ```apk add --update --no-cache python3 && ln -sf python3 /usr/bin/python```) and launch a python minimal hello world from the container. 
2. Experiment with environmental variables. Create a new container from the getting-started image with an environment variable (hint: use the ```-e``` flag of docker run, e.g. ```docker run -d -p 8081:80 -e GREETING=ciao docker/getting-started```). Install python also on this container (as we said in the previous lessons, the edits we do in a container remain in the container layer - parent image layers are read-only) and try to print out the environment variable from a python script (see <a target="_blank" rel="noopener noreferrer" href="https://docs.python.org/3/library/os.html">the os python standard module</a>).  
<div class="lesson-nav">
    <div>
    Previous: <a href="/SoftwareArchitectures_2025/docker/introduction">Docker - Introduction</a>
    </div>
    <div>
    Next: <a href="/SoftwareArchitectures_2025/docker/cli-and-dockerfile">Docker - CLI and Dockerfile</a>  
    </div>
</div>