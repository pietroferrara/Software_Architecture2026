---
title: 3. CLI and Dockerfile
category: 01. Docker
order: 3
---
<h2>Contents</h2>
* toc
{:toc}
In the previous sections, we talked about Docker and we saw how we can download an image and launch a container.  
Here, we are going to explain how to interact with the Docker daemon using the Docker CLI and how we can create our custom images.

## The Docker CLI
The Docker Engine is shipped out with a docker CLI client application.
You can interact with docker using the *docker* command:

{% highlight bash %}
~/Projects/sw-arch/docker docker

Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Common Commands:
  run         Create and run a new container from an image
  exec        Execute a command in a running container
  ps          List containers
  build       Build an image from a Dockerfile
  bake        Build from a file
  pull        Download an image from a registry
  push        Upload an image to a registry
  images      List images
  login       Authenticate to a registry
  logout      Log out from a registry
  search      Search Docker Hub for images
  version     Show the Docker version information
  info        Display system-wide information

Management Commands:
  builder     Manage builds
  buildx*     Docker Buildx
  compose*    Docker Compose
  container   Manage containers
  context     Manage contexts
  image       Manage images
  manifest    Manage Docker image manifests and manifest lists
  model*      Docker Model Runner
  network     Manage networks
  plugin      Manage plugins
  system      Manage Docker
  volume      Manage volumes

Swarm Commands:
  swarm       Manage Swarm

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  wait        Block until one or more containers stop, then print their exit codes

Global Options:
      --config string      Location of client config files (default
                           "/home/user/.docker")
  -c, --context string     Name of the context to use to connect to the
                           daemon (overrides DOCKER_HOST env var and
                           default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host string        Daemon socket to connect to
  -l, --log-level string   Set the logging level ("debug", "info",
                           "warn", "error", "fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default
                           "/home/user/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default
                           "/home/user/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default
                           "/home/user/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Run 'docker COMMAND --help' for more information on a command.

For more help on how to use Docker, head to https://docs.docker.com/go/guides/
{% endhighlight %}

Docker CLI commands are self-explanatory: however, we will see briefly the most important ones.
### Show all the downloaded images
You can get a list of all the images that you have downloaded with the command **docker images** (or **docker image ls**):
{% highlight bash %}
~/Projects/sw-arch/docker docker images
IMAGE                             ID             DISK USAGE   CONTENT SIZE   EXTRA
docker/getting-started:latest     d79336f4812b       73.9MB         21.4MB        
httpd:latest                      c842ac797bd1        177MB         47.7MB        
node:24-alpine                    ebfe2f904627        242MB           62MB        
{% endhighlight %}
### Download an image from the Docker Hub
What about downloading an image? Just type <b>docker pull *&lt;name-of-image:tag&gt;*</b> (example: **docker pull python:3.13-alpine**).  
Tag permits you to specify which version of the Docker image you want to use. If you don't specify a tag, the latest available image will be pulled.
You can see all the available tags for an image directly from the Docker Hub (for example, for python: <a target="_blank" rel="noopener noreferrer" href="https://hub.docker.com/_/python/tags">hub.docker.com/_/python/tags</a>).
{% highlight bash %}
~/Projects/sw-arch/docker docker pull python:3.13-alpine
3.13-alpine: Pulling from library/python
82c48d7f22df: Pulling fs layer
48f6a2588427: Pulling fs layer
6f4c23997e2d: Pulling fs layer
48f6a2588427: Download complete
82c48d7f22df: Download complete
582f0d637aa6: Download complete
6f4c23997e2d: Download complete
48f6a2588427: Pull complete
3dd514ae25f6: Download complete
6f4c23997e2d: Pull complete
82c48d7f22df: Pull complete
Digest: sha256:2d9aefe2fef018a7eb2c13064c89c71929800fd2e5dccdbf52ea5da5bb8d929a
Status: Downloaded newer image for python:3.13-alpine
docker.io/library/python:3.13-alpine
~/Projects/sw-arch/docker docker images
IMAGE                             ID             DISK USAGE   CONTENT SIZE   EXTRA
docker/getting-started:latest     d79336f4812b       73.9MB         21.4MB        
httpd:latest                      c842ac797bd1        177MB         47.7MB        
node:24-alpine                    ebfe2f904627        242MB           62MB        
python:3.13-alpine                2d9aefe2fef0       79.4MB         20.1MB        
~/Projects/sw-arch/docker 
{% endhighlight %}
As we mentioned before, an image is composed of layers, that are images themselves. When you download an image, you download the layers that compose that image and this is visible by the logs of the docker pull command (i.e., 82c48d7f22df, 48f6a2588427, 6f4c23997e2d are some of the layers that compose this image). This permits you to avoid redownloading a layer if you have already pulled it for another image.  

### List, create, and run containers
To create a container from an image, you can use the docker create command:
{% highlight bash %}
~/Projects/sw-arch/docker docker create -p 8080:80 httpd 
597372a315efb6465d2900a2cabb7b570b6cee884cf87781927443365435cd75
{% endhighlight %}
With the -p flag you can specify the port mapping: in the example, we map port 8080 of the host to port 80 in the container.
If you want to see all the available containers in your docker environment, you can use docker ps -a command (without the -a flag, docker ps shows only the running containers):
{% highlight bash %}
~/Projects/sw-arch/docker docker ps -a
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS                       PORTS                  NAMES
597372a315ef   httpd                           "httpd-foreground"       50 seconds ago   Created                                             tender_banzai
7e1bd280a489   node:18                         "docker-entrypoint.s…"   3 days ago       Exited (0) 3 days ago                               happy_lalande
c9338f41f0dc   docker/getting-started          "/docker-entrypoint.…"   3 days ago       Exited (255) 6 minutes ago   0.0.0.0:80->80/tcp     great_lehmann
7d7fb37297b2   docker/getting-started:latest   "/docker-entrypoint.…"   4 days ago       Up About a minute            0.0.0.0:1234->80/tcp   romantic_elgamal
c39fccc5e710   docker/getting-started:latest   "/docker-entrypoint.…"   4 days ago       Exited (0) 3 days ago  
~/Projects/sw-arch/docker
{% endhighlight %}
To start a container, you can use the docker start command, passing the ID of the desired container:
{% highlight bash %}
~/Projects/sw-arch/docker docker start 597372a315ef 
597372a315ef
~/Projects/sw-arch/docker
{% endhighlight %}
You can also create and start a container in a single step, by using docker run:
{% highlight bash %}
~/Projects/sw-arch/docker docker run -d -p 8081:80 httpd 
5a3280c5fc90340419f8f17603580e1c46fb5349a786f0ff837908f0ccfb623b
{% endhighlight %}
### Interact with a running container
As we saw in the previous section, you can read the logs of a container with **docker logs** and run a command inside a running container with **docker exec** (with the -it flags, you get an interactive terminal):
{% highlight bash %}
~/Projects/sw-arch/docker docker logs 5a3280c5fc90
...
~/Projects/sw-arch/docker docker exec -it 5a3280c5fc90 bash
{% endhighlight %}
Containers can be stopped with **docker stop** and deleted with **docker rm**.
### Build an image
We can use the docker build command to build an image. We will explain this command in the next section.
### Other commands
An exhaustive list of all the Docker CLI commands can be found <a target="_blank" rel="noopener noreferrer" href="https://docs.docker.com/reference/cli/docker/">here</a>.
## The Dockerfile
But how we can create an image from scratch? In Docker exists a special file, called Dockerfile, that permits the definition of an image in a script way. Let's see how it works.
The first step involves creating a new text file named Dockerfile, without extension. In this file, we are going to describe all the step-by-step commands that the Docker Engine needs to run to assemble a Docker image.  
We are going to explain how a Dockerfile works by using a simple web application (a to-do list written in NodeJS) provided by Docker.   
So, first of all, download the application:
{% highlight bash %}
~/Projects/sw-arch/docker git clone https://github.com/docker/getting-started-app.git
~/Projects/sw-arch/docker cd getting-started-app
~/Projects/sw-arch/docker/getting-started-app ls
package.json  package-lock.json  README.md  spec  src
{% endhighlight %}
Then, inside the getting-started-app folder, create an empty file named Dockerfile and insert the following lines:
{% highlight docker %}
FROM node:24-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --omit=dev
COPY src ./src
CMD ["node", "src/index.js"]
{% endhighlight %}
What do these commands mean? Let's see one by one.  
1. ```FROM node:24-alpine```: as you can notice, we downloaded a node application. The FROM command tells us the base image: in this case, since we are dealing with a node app, we want to start with a node image (version 24, based on Alpine Linux).  
2. ```WORKDIR /app```: this says that the default working directory for the next commands is the /app folder. If the folder does not exist in the image, this command will create it.
3. ```COPY package.json package-lock.json ./```: this command permits us to copy files from our machine to the Docker image. The syntax is COPY &lt;src&gt; &lt;dest&gt;: in our case, we copy the two files that describe the dependencies of our project into the current workdir of the image that we are going to create.
4. ```RUN npm ci --omit=dev```: this command launches npm. npm is a package manager for node, it permits the installation of all the necessary external dependencies for our project, defined inside the package.json file (--omit=dev skips the dependencies needed only during development).
5. ```COPY src ./src```: this command copies the source code of the application into the image.
6. ```CMD ["node", "src/index.js"]```: tells what command needs to be launched when we start a container created from the current image. For our case, we are going to start the node server.

Why do we copy package.json first, and the source code later? We will see it in a moment, when we will talk about the build cache.
### RUN and CMD: what's the difference?
RUN and CMD commands seem very similar. The main difference is that RUN is an image-build step. That is to say, all commands launched with RUN will be persistent in the image. We could have multiple RUN commands in a Dockerfile, and these commands are layered on top of one another to build the final image.  
CMD, instead, is the command launched by default when we start the container. The Dockerfile will just use the last CMD defined.  
In other words: RUN commands will be executed once during the creation of the image. The CMD command will be executed every time we start a container.
### Build the image
To build the image we use the docker build command. Go into the folder in which the Dockerfile lives and launch the next command:  
{% highlight bash %}
docker build -t simple-node-app .
{% endhighlight %}
This command creates a new image executing the commands specified in the Dockerfile: first, it will fetch the node:24-alpine image from the web. Then, it will copy the package.json and package-lock.json files inside the /app folder, it will run npm to install all the dependencies, and finally it will copy the source code.
The -t flag permits to tag and name the image.  
The last point (.) at the end of the command tells Docker to fetch the Dockerfile from the current directory.  
Now, if you list the images, you should see the newly created image:
{% highlight bash %}
~/Projects/sw-arch/docker/getting-started-app docker images simple-node-app
IMAGE                    ID             DISK USAGE   CONTENT SIZE   EXTRA
simple-node-app:latest   256ac2470812        315MB         80.7MB        
{% endhighlight %}
Create and run a container for our image:
{% highlight bash %}
docker run -d -p 3200:3000 simple-node-app
{% endhighlight %}
With the flag -d we are going to tell Docker to run the container in detached mode, while with -p we can specify the port mapping: in our case, we are mapping the 3200 port of our host to port 3000 of the container.
The application is then available at <a target="_blank" rel="noopener noreferrer" href="http://localhost:3200">localhost:3200</a>:
![Docker images]({{ site.baseurl }}/images/docker_cli_4.png)
### The build cache
Each instruction of the Dockerfile creates a layer of the image. When you rebuild an image, Docker reuses the layers that did not change (this is called the **build cache**): a layer is rebuilt only if its instruction or the files it copies have changed, and in that case all the following layers are rebuilt too.  
This is why we copied package.json before the source code: if we change only a source file and we rebuild the image, the dependencies are not installed again:
{% highlight bash %}
~/Projects/sw-arch/docker/getting-started-app docker build -t simple-node-app .
...
#6 [3/5] COPY package.json package-lock.json ./
#6 CACHED
#7 [2/5] WORKDIR /app
#7 CACHED
#8 [4/5] RUN npm ci --omit=dev
#8 CACHED
#9 [5/5] COPY src ./src
#9 DONE 0.5s
...
{% endhighlight %}
Installing the dependencies is usually the slowest step of the build, so the order of the instructions matters: put the instructions that change rarely at the top of the Dockerfile, and the ones that change often at the bottom.
### The .dockerignore file
When you build an image, Docker sends the content of the folder (the build context) to the daemon. With a .dockerignore file, placed next to the Dockerfile, you can exclude files and folders that should not end up in the image. Its syntax is similar to the .gitignore one. The application that we downloaded already contains one (it is a hidden file: use ls -a to see it):
{% highlight bash %}
~/Projects/sw-arch/docker/getting-started-app cat .dockerignore
node_modules
Dockerfile
{% endhighlight %}
In this way, if you have installed the dependencies on your machine, the local node_modules folder (that can be very big) is not sent to the daemon, and it cannot end up in the image by mistake, for example with an instruction like ```COPY . .``` (the dependencies are installed inside the image by npm).
### Other Dockerfile commands
Here we see briefly three other important Dockerfile commands: a complete list can be found <a target="_blank" rel="noopener noreferrer" href="https://docs.docker.com/reference/dockerfile/">here</a>.
1\. **EXPOSE**: informs Docker that the container listens on specified network ports at runtime, and permits to specify also if the port listens on TCP or UDP protocol (default TCP if the protocol is omitted). For example:  
{% highlight docker %}
EXPOSE 80/udp
EXPOSE 80/tcp
EXPOSE 40
{% endhighlight %}
This tells that the container listens on port 80 both in UDP and TCP, and on port 40 in TCP. Note that EXPOSE works as documentation: to reach the port from your machine, you still need to map it with the -p flag.
2\. **ENV**: sets an environment variable in the container. For example, try to create this Docker image:
{% highlight docker %}
FROM ubuntu
ENV TEST_ENV_VAR="HELLO WORLD"
CMD ["sleep", "3600"]
{% endhighlight %}
And run a container from it. Then, open a terminal inside the container and print out the content of the environment variable TEST_ENV_VAR:
{% highlight bash %}
~/Projects/sw-arch/docker docker exec -it <container-id> sh
# echo $TEST_ENV_VAR
HELLO WORLD
{% endhighlight %}
3\. **ENTRYPOINT**: like CMD, it defines the command launched when we start the container. The difference is that the arguments passed to docker run replace the CMD, but they are appended to the ENTRYPOINT. They are often used together: ENTRYPOINT defines the executable, and CMD its default arguments. For example:
{% highlight docker %}
FROM alpine
ENTRYPOINT ["echo", "Hello"]
CMD ["world"]
{% endhighlight %}
{% highlight bash %}
~/Projects/sw-arch/docker docker build -t hello .
~/Projects/sw-arch/docker docker run --rm hello
Hello world
~/Projects/sw-arch/docker docker run --rm hello students
Hello students
{% endhighlight %}
(The --rm flag deletes the container when it stops.)
## Exercises
1. Create a **Dockerfile** that prints "installing!" when it is installed and prints "running!" each time it is launched.  
Hint: by default, docker build shows the output of the RUN commands only briefly, and a cached step is not executed again. To see the output, build the image with ```docker build --progress=plain --no-cache -t <name> .```  
<div class="lesson-nav">
  <div>
  Previous: <a href="{{ site.baseurl }}/docker/getting-started">Docker - Getting Started</a>
  </div>
  <div>
  Next: <a href="{{ site.baseurl }}/docker/volumes">Docker - Volumes</a>  
  </div>
</div>
