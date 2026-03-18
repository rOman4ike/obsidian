
# Refs

1. https://courses.mooc.fi/org/uh-cs/courses/devops-with-docker/chapter-1

# Methods

1. `docker container run`, `docker run`, `-d`
2. `docker image ls`
3. `docker container ls`, `-a`, `docker ps`,
4. `docker image rm`
5. `docker container rm`,
6. `docker container prune`, `docker image prune`, `docker system prune`
7. `docker image pull`
8. `docker container stop`

# Questions

1. Docker daemon
2. Docker Hub
3. Docker Engine
4. Interaction between containers
5. Dangling images

# Raw Info

> "Docker is a set of platform as a service (PaaS) products that use OS-level virtualization to deliver software in packages called containers." - [from Wikipedia(opens in a new tab)](https://en.wikipedia.org/wiki/Docker_\(software\)).

> Docker is a set of tools to deliver software in containers.

> Containers are packages of software.

![[Pasted image 20260227122729.png]]

Benefits from containers​
Containers package applications. Sounds simple, right? To illustrate the potential benefits let's talk about different scenarios.
1. Scenario 1: Works on my machine[​](http://localhost:3000/part-1/section-1#scenario-1-works-on-my-machine)
2. Scenario 2: Isolated environments[​](http://localhost:3000/part-1/section-1#scenario-2-isolated-environments)
3. Scenario 3: Development[​](http://localhost:3000/part-1/section-1#scenario-3-development)
4. Scenario 4: Scaling[​](http://localhost:3000/part-1/section-1#scenario-4-scaling)

Virtual Machines are not the same as Containers - they solve different problems.

![[Pasted image 20260227123435.png]]

Virtual Machines (VMs) run on a [hypervisor(opens in a new tab)](https://en.wikipedia.org/wiki/Hypervisor), which virtualizes the physical hardware. Each VM includes a full operating system (OS) along with the necessary binaries and libraries, making them heavier and more resource-intensive. Containers, on the other hand, share the host OS kernel and only package the application and its dependencies, resulting in a more lightweight and efficient solution.

## Image and containers[​](http://localhost:3000/part-1/section-1#image-and-containers)

Since we already know what containers are it's easier to explain images through them: Containers are instances of images. A basic mistake is to confuse images and containers.

Cooking metaphor:

Think of a container as a ready-to-eat meal that you can simply heat up and consume. An image, on the other hand, is the recipe _and_ the ingredients for that meal.

So just like how you need a recipe and ingredients to make a meal, you need an image and a container runtime (Docker engine) to create a container. 

In short, an image is like a blueprint or template and the building material, while a container is an instance of that blueprint or template.

A Docker image is a file. An image never changes; you can not edit an existing file. Creating a new image happens by starting from a base image and adding new layers to it. We will talk about layers later, but you should think of images as immutable, they can not be changed after they are created.

List all your images with `docker image ls`

Containers are created from images, so when we ran hello-world twice we downloaded one image and created two separate containers from the single image.

Well then, if images are used to create containers, where do images come from? This image file is built from an instructional file named Dockerfile that is parsed when you run docker image build.

Dockerfile is a file that is by default called Dockerfile, that looks something like this

```
FROM <image>:<tag>

RUN <install some dependencies>

CMD <command that is executed on `docker container run`>
```

and is the instruction set for building an image. We will look into Dockerfiles later when we get to build our own images.

Containers contain the application and what is required to execute it (dependencies); and you can start, stop and interact with them. They are isolated environments in the host machine with the ability to interact with each other and the host machine itself via defined methods (TCP/UDP).

List all your containers with `docker container ls`

Without `-a` flag it will only print running containers. The hello-worlds we ran already exited.

## Docker CLI basics[​](http://localhost:3000/part-1/section-1#docker-cli-basics)

We are using the command line to interact with the "Docker Engine" that is made up of 3 parts:
1. command line interface (CLI) client
2. a REST API
3. Docker daemon

When you run a command, e.g. docker container run, behind the scenes the CLI client sends a request to the Docker daemon through the REST API. The Docker daemon takes care of images, containers and other resources.

Let's remove the image since we will not need it anymore, `docker image rm hello-world` sounds about right. However, this should fail with the following error:

```pgsql
$ docker image rm hello-world
  Error response from daemon: conflict: unable to remove repository reference "hello-world" (must force) - container <container ID> is using its referenced image <image ID>
```

This means that a container that was created from the image _hello-world_ still exists and that removing _hello-world_ could have consequences. So before removing images, you should have the referencing container removed first. Forcing is usually a bad idea, especially as we are still learning.

Run `docker container ls -a` to list all containers again.

Run `docker container ls -a` to list all containers again.

```java
$ docker container ls -a
  CONTAINER ID   IMAGE           COMMAND        CREATED          STATUS                      PORTS     NAMES
  b7a53260b513   hello-world     "/hello"       35 minutes ago   Exited (0) 35 minutes ago             brave_bhabha
  1cd4cb01482d   hello-world     "/hello"       41 minutes ago   Exited (0) 41 minutes ago             vibrant_bell
```

Notice that containers have a _CONTAINER ID_ and _NAME_. The names are currently autogenerated. When we have a lot of different containers, we can use grep (or another similar utility) to filter the list:

```mel
$ docker container ls -a | grep hello-world
```

Let's remove the container with `docker container rm` command. It accepts a container's name or ID as its arguments.

Notice that the command also works with the first few characters of an ID. For example, if a container's ID is 3d4bab29dd67, you can use `==docker container rm 3d==` to delete it. Using the shorthand for the ID will not delete multiple containers, so if you have two IDs starting with 3d, a warning will be printed, and neither will be deleted. You can also use multiple arguments: `docker container rm id1 id2 id3`

If you have hundreds of stopped containers and you wish to delete them all, you should use `docker container prune`. Prune can also be used to remove "dangling" images with `docker image prune`. Dangling images are images that don't have a name and are not used. They can be created manually and are automatically generated during build. Removing them just saves some space.

And finally you can use `docker system prune` to clear almost everything. We aren't yet familiar with the exceptions that `docker system prune`doesn't remove.

You can also use the image pull command to download images without running them: so docker image pull hello-world would bring the image back to our computer.

Let's try starting a new container:

```routeros
$ docker run nginx
```

With some containers the command line appears to freeze after pulling and starting the container. This might be because that particular container is now running in the current terminal, blocking the input. You can observe this with `docker container ls` from another terminal. In this situation one can exit by pressing `control + c` and try again with the `-d` flag.

The `-d` flag starts a container _detached_, meaning that it runs in the background. The container can be seen with

Now if we try to remove it, it will fail:

```routeros
$ docker container rm blissful_wright
  Error response from daemon: You cannot remove a running container c7749cf989f61353c1d433466d9ed6c45458291106e8131391af972c287fb0e5. Stop the container before attempting removal or force remove
```

We should first stop the container using `==docker container stop blissful_wright==`, and then use `rm`.

Forcing is also a possibility and we can use `==docker container rm --force blissful_wright==` safely in this case. Again for both of them instead of name we could have used the ID or parts of it, e.g. c77.

### Most used commands[​](http://localhost:3000/part-1/section-1#most-used-commands)

|command|explain|shorthand|
|---|---|---|
|`docker image ls`|Lists all images|`docker images`|
|`docker image rm <image>`|Removes an image|`docker rmi`|
|`docker image pull <image>`|Pulls image from a docker registry|`docker pull`|
|`docker container ls -a`|Lists all containers|`docker ps -a`|
|`docker container run <image>`|Runs a container from an image|`docker run`|
|`docker container rm <container>`|Removes a container|`docker rm`|
|`docker container stop <container>`|Stops a container|`docker stop`|
|`docker container exec <container>`|Executes a command inside the container|`docker exec`|


