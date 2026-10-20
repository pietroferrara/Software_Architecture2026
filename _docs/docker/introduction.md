---
title: 1. Introduction 
category: 01. Docker
order: 1
---
![Docker logo]({{ site.baseurl }}/images/docker_logo.jpeg)
<h2>Contents</h2>
* toc
{:toc}
<center><iframe width="100%" height="315" src="https://www.youtube.com/embed/2xZNUCwUpsU?si=VFF-zZUAiBiRV932" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe></center>
## Install Docker Desktop
Download **Docker Desktop** from [here](https://www.docker.com/products/docker-desktop/). Then, run the installer and follow the onscreen instructions. This will install the Docker Engine, which is the core of the Docker platform, and an easy-to-use Graphical User Interface (GUI), i.e. Docker Desktop.
![Docker Desktop]({{ site.baseurl }}/images/docker_desktop.png)
A couple of notes depending on your operating system:
1. **Windows**: Docker Desktop runs on top of the <a target="_blank" rel="noopener noreferrer" href="https://learn.microsoft.com/en-us/windows/wsl/install">Windows Subsystem for Linux (WSL 2)</a>. If the installer asks you to enable or update WSL 2, just follow the instructions.
2. **Linux**: you can install Docker Desktop, or just the Docker Engine (see <a target="_blank" rel="noopener noreferrer" href="https://docs.docker.com/engine/install/">here</a> for your distribution). If you install only the Engine, add your user to the docker group, so you can run docker commands without sudo: ```sudo usermod -aG docker $USER```. Then, log out and log in again (the new group is applied only to new sessions).
## The Docker Engine
The Docker Engine is shipped out with a Command-Line Interface (CLI), and a server. The latter component runs as a root-privilege background process (daemon). You can interact with the server using the CLI client or Docker Desktop. The daemon creates and manages Docker objects, such as images and containers. 
Launching Docker Desktop will automatically start the underlying daemon process.

## Containers

As you may have learned from the video, Docker is a tool that permits running an application in an isolated environment, enabling the capability to separate applications from infrastructures. This environment is named **container** and can be seen as a sandboxed process running on a machine. In a single machine, you can have multiple containers, each of one containing one or more applications. According to docker documentation, "a container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another". 
Docker containers are:
1. Industry standard: they could be portable anywhere.
2. Lightweight: containers are not required to run a full OS per application, since all the containers share the machine's OS system kernel.
3. Isolated: each container runs separated from the other containers and from the rest of the system. Note that isolation is not the same as full security: all the containers share the host's kernel, and the Docker daemon runs with root privileges. This is why giving a user access to Docker (e.g., adding it to the docker group) is equivalent to giving it root access.  
![Docker container]({{ site.baseurl }}/images/docker_container.webp)
Docker automates the deployment of applications into containers, providing a set of tools and mechanisms that allow its management.


## Images
A container is a runnable instance of an **image**. Images are the building part of Docker's life cycle, and they are made up of filesystems layered over each other using a union mount. Every layer of an image is mounted in read-only and is itself an image: the image a layer is built upon is called its parent image, whereas the bottom-most one is called the base image.
When we create a container from an image, Docker will add a read-write filesystem on top of all the image layers. This layer runs the application that we want to "Dockerize". When Docker first starts a container, this read-write layer is empty. Changes are applied only to this layer: for example, if the running application needs to change a file of the parent image, this file is copied into the container layer (i.e., the read-write layer), shadowing the read-only file.  
To better understand this concept, just observe the image below:
![Docker image layers]({{ site.baseurl }}/images/docker_image_layers.svg)

This figure depicts the layered structure of an image: starting from the bottom, we have the kernel of the host operating system. This is not part of the image: a container does not boot its own kernel, but all the containers running on a machine share the kernel of the host. This is why containers are much lighter than virtual machines.

Then, we will find the Base Image: this represents the starting point of our layered image. Usually, base images are basic minimal Linux distributions, for example, Ubuntu, Redhat, Centos, Alpine, or Debian.  

The next layer is the parent image of our image: simply it adds emacs (a CLI text editor) to our base image, Debian. Our image is built on top of Debian + emacs and provides an Apache HTTP server. The last layer is the read-write layer of the container, and inside of it lives the actual app.  
<div class="lesson-nav">
    <div>
    Next: <a href="{{ site.baseurl }}/docker/getting-started">Docker - Getting Started</a>
    </div>
</div>