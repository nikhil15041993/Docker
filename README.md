# Docker – Removing Dangling

## Dangling  Docker Images

Dangling images are created when we overwrite them with a new image of the same name and tag.

Let's look at a small example of how updating an image will lead to a dangling image. Below is a simple Dockerfile:

```
FROM ubuntu:latest
CMD ["echo", "Hello World"]
```
Let's build this image:

```
docker build -t my-image .
```
We can verify that the image is created by running the below command:

```
docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
my-image     latest    7ed6e7202eca   32 seconds ago   72.8MB
ubuntu       latest    825d55fb6340   6 days ago       72.8MB
```
Suppose we add a little change to the Dockerfile:

```
FROM ubuntu:latest
CMD ["echo", "Hello, World!"]
```
Let's rebuild the image using the same command as before and list the images again:

```
docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
my-image     latest    da6e74196f66   4 seconds ago        72.8MB
<none>       <none>    7ed6e7202eca   About a minute ago   72.8MB
ubuntu       latest    825d55fb6340   6 days ago           72.8MB
```
The build created a new my-image image. As we can see, the old image is still there, but now it's dangling. Its name and tag are set to <none>:<none>.

Please note that if we did not make a change to the Dockerfile, the image would not be rebuilt while running the build command. It will be reused from the cache.





## Unused Docker Images

Unused images are images that do not have a running or stopped container associated with them.

Examples of unused images are:

* Images pulled from a registry but not used in any container yet
* Any image whose containers have been removed
* An image tagged with an older version and not being used anymore
* All dangling images



## Docker Image Prune

If we do not want to find dangling images and remove them one by one, we can use the docker image prune command. This command removes all dangling images.

If we also want to remove unused images, we can use the -a flag.

Let's run the below command:

```
docker image prune -a

WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y
```
The command will return the list of image IDs that were removed and the space that was freed.




## Docker System Prune

Finding and removing all unused objects can be tedious. To make things easier, we can use the ``` docker system prune ``` command.

This command, by default, will remove the below objects:

* Stopped containers – containers with a stopped status
* Networks that are not being used by at least one container
* Dangling images
* Dangling build cache – the build cache that was supporting dangling images

Additionally, we can add the -a flag to remove all unused containers, images, networks, and the entire build cache. This is useful when we want to free up a lot of space.

Let's look at an example of this:

```
docker system prune -a

WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all images without at least one container associated to them
  - all build cache

Are you sure you want to continue? [y/N]
```
