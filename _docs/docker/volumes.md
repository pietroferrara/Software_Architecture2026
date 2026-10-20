---
title: 4. Volumes 
category: 01. Docker
order: 4
---
<h2>Contents</h2>
* toc
{:toc}
## A Glimpse of our Node Application
In the previous section, we saw how to create a Dockerfile and we deployed our first application inside a container.
This demo application is a simple to-do list. Let's try to add something:
![Docker images]({{ site.baseurl }}/images/docker_volumes_1.png)
As you can notice, the inserted data seems persistent (spoiler: they are not): try to refresh the page, or to restart the container. The items that we inserted remain. Let's see how it's done by looking at the source code of the application. Even if we don't know how nodeJS works (it is not part of the course), we can see that underlying the application there is a database (src/persistence folder), and looking at the src/persistence/index.js file we know that, since we didn't have set an environment variable MYSQL_HOST we are using SQLite. SQLite stores data in /etc/todos/todo.db (line 3 of sqlite.js):
![Docker images]({{ site.baseurl }}/images/docker_volumes_2.png)
We can prove it just by inspecting the todo.db file from inside the container (the strings command prints the readable text contained in a binary file; use docker ps to get the ID of your container):
{% highlight bash %}
~/Projects/sw-arch/docker/getting-started-app docker exec <container-id> strings /etc/todos/todo.db
SQLite format 3
)tabletodo_itemstodo_items
CREATE TABLE todo_items (id varchar(36), name varchar(255), completed boolean)
1f513b74-cf03-4e97-815e-e450c20a9a2ebuy milk:
7d8b1d1d-044e-4365-9f79-3e69810e3ffbstudy for the exam
{% endhighlight %}

### Updating the App

Now, suppose that you need to update the application: we have been asked to change the text shown when the to-do list is empty. In the src/static/js/app.js file, update line 56 in this way:
{% highlight diff %}
- <p className="text-center">No items yet! Add one above!</p>
+ <p className="text-center">You have no todo items yet! Add one above!</p>
{% endhighlight %}
Then, rebuild the image, delete the old container, and start a new one from the updated image:
{% highlight bash %}
~/Projects/sw-arch/docker/getting-started-app docker build -t simple-node-app .
~/Projects/sw-arch/docker/getting-started-app docker rm -f <container-id>
~/Projects/sw-arch/docker/getting-started-app docker run -d -p 3200:3000 simple-node-app
{% endhighlight %}
(Why do we need to delete the old container? Try to start the new one without deleting the old one, and look at the error.)  
Go to <a target="_blank" rel="noopener noreferrer" href="http://localhost:3200">localhost:3200</a>: you should see the new text. But, as you can imagine, our data is gone (Why?).

### Where are my data?
Our data is gone because it lives inside a container, and updating an app requires building a new container. This is not optimal: suppose that our app is widely used around the world and therefore
we have multiple servers (and multiple containers) that host the application for load-balancing purposes. We want the data to persist among containers and among different updates. What we can do?
## Docker Volumes
To ensure the persistence of data, volumes come to our help.  
A volume is a persistent storage area, created and managed by Docker, that can be mounted into one or more containers. Its content lives outside the container's writable layer, so it survives when containers are removed or recreated. (Mounting a specific directory of our host machine into a container is a different mechanism, called a *bind mount*.) We can create a volume with the next command:
{% highlight bash %}
docker volume create <name>
{% endhighlight %}
Where *&lt;name&gt;* is the name of the volume. So let's create a volume for our application:
{% highlight bash %}
~/Projects/sw-arch/docker/getting-started-app docker volume create todolist-db      
todolist-db
~/Projects/sw-arch/docker/getting-started-app docker volume ls
DRIVER    VOLUME NAME
local     todolist-db
~/Projects/sw-arch/docker/getting-started-app docker volume inspect todolist-db
[
    {
        "CreatedAt": "2026-10-13T09:23:48Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/todolist-db/_data",
        "Name": "todolist-db",
        "Options": null,
        "Scope": "local"
    }
]
{% endhighlight %}
As you can see from the Mountpoint, the data of the volume is stored in a folder managed by Docker (on macOS and Windows, this folder lives inside the virtual machine used by Docker Desktop, so you cannot access it directly). If you use Docker Desktop, you should see the volume also inside the Volumes tab:
![Docker images]({{ site.baseurl }}/images/docker_volumes_5.png)
Now we want to our app use this volume. In order to do so, destroy the container and create it by launching the next command:
{% highlight bash %}
~/Projects/sw-arch/docker/getting-started-app docker run -d -p 3200:3000 -v todolist-db:/etc/todos simple-node-app
aa68638a2fa01e8f499a916f5f8bd2f0d59c25c73ef3ebaeeb9354c8e5a50490
~/Projects/sw-arch/docker/getting-started-app 
{% endhighlight %}
The -v flag permits to specify the volume: the string "todolist-db:/etc/todos" means that the todolist-db volume is mounted in /etc/todos, so all the content that the application generates inside the /etc/todos folder will be stored persistently inside the volume. Therefore, if we rebuild the image and recreate the container, data will not be lost. Try to do some tests: add some entries in the todo list, then destroy and recreate the container. You should note the persistence of the information. Try also to launch multiple containers with different binding ports and see with your eyes.
## Bind Mounts
Volumes are the best choice to store the data of our application. But there is another type of mount, very useful during development: the **bind mount**. With a bind mount, we mount a specific folder of our machine inside the container: the container sees the files of the host folder, and every change is immediately visible on both sides.  
The syntax is the same of volumes, but instead of the name of a volume we specify a path of the host (it must be an absolute path, so we use $(pwd), i.e. the current folder). For example, we can run our application from the source code of our machine, without building an image:
{% highlight bash %}
~/Projects/sw-arch/docker/getting-started-app docker run -d -p 3200:3000 -w /app -v "$(pwd):/app" node:24-alpine sh -c "npm install && npm run dev"
~/Projects/sw-arch/docker/getting-started-app docker logs -f <container-id>
...
[nodemon] starting `node src/index.js`
Using sqlite database at /etc/todos/todo.db
Listening on port 3000
{% endhighlight %}
Here, we mount the current folder (the source code of the application) in the /app folder of the container (-w /app sets the working directory), and we start the application with nodemon, a tool that restarts the application each time a file changes. Now, try to edit a file of the src folder from your editor: the application restarts automatically inside the container, without rebuilding the image.  
Note that npm install, launched inside the container, creates the node_modules folder in your project folder (because of the bind mount): on Linux, it will be owned by root.
<div class="lesson-nav">
    <div>
        Previous: <a href="{{ site.baseurl }}/docker/cli-and-dockerfile">Docker - CLI and Dockerfile</a>
    </div>
</div>