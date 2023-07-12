## Set up a Private Registry
we first need to make changes in the default configuration of the Docker daemon.

### Configure a Private Docker Registry
In Docker, we can set up a registry by running a container of a registry image. Before we move forward, let's first update the default configuration of our Docker install.

Add the following configuration in the /etc/docker/daemon.json:

```
{
    "insecure-registries":[
        "localhost:5000"
    ]
}
```
In the above JSON, we've added localhost with port 5000 in the “insecure-registries” property. To apply the above changes, let's reload the Docker daemon using the command line:
```
$ sudo systemctl daemon-reload
```
Now, we'll restart the Docker service:

```
sudo systemctl restart docker
```
### Run a Private Docker Registry
To run a private registry, we have to pull a registry image stored on the public Docker Hub:
```
$ docker pull registry
Using default tag: latest
latest: Pulling from library/registry
2408cc74d12b: Pull complete 
...
fc30d7061437: Pull complete 
Digest: sha256:bedef0f1d248508fe0a16d2cacea1d2e68e899b2220e2258f1b604e1f327d475
Status: Downloaded newer image for registry:latest
docker.io/library/registry:latest
```

We can also pull a specific version of the registry. Let's now verify the registry image using the docker images command:
```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
registry            latest              773dbf02e42e        21 hours ago        24.1MB
```

Now, let's run a Docker container using the registry image:
```
$ docker run -itd -p 5000:5000 --name baeldung-registry registry
e2d09cd3a5ef9c88e17e0393f7125b6eeffad175fa0ce69fa3daa7803a0b3067 
```
The internal server of the baeldung-registry container uses port 5000. Therefore, we exposed the 5000 port on the host:

```
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
e2d09cd3a5ef        registry            "/entrypoint.sh /etc…"   3 minutes ago       Up 2 minutes        0.0.0.0:5000->5000/tcp   baeldung-registry
```

### Push an Image to the Private Registry
To push an image to the private registry, let's first pull the latest centos image from the public Docker registry:

```
$ docker pull centos
Using default tag: latest
latest: Pulling from library/centos
a1d0c7532777: Pull complete 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
docker.io/library/centos:latest
```

First, we'll tag the centos image, and later push it to the private docker registry. Here, we'll tag it to localhost:5000/baeldung-centos:
```
docker tag centos:latest localhost:5000/baeldung-centos
```

Now, let's check out all the images on the host machine:
```
$ docker images
REPOSITORY                          TAG                 IMAGE ID            CREATED             SIZE
registry                            latest              773dbf02e42e        22 hours ago        24.1MB
localhost:5000/baeldung-centos   latest              5d0da3dc9764        8 months ago        231MB
centos     
```

Let's check out the command to push the image to the docker private registry:
```
$ docker push localhost:5000/baeldung-centos
The push refers to repository [localhost:5000/baeldung-centos]
74ddd0ec08fa: Pushed 
latest: digest: sha256:a1801b843b1bfaf77c501e7a6d3f709401a1e0c83863037fa3aab063a7fdb9dc size: 529
```
With the above command, we have successfully pushed the baeldung-centos image to the private registry, which is locally set up on port 5000. 

### Pull an Image From the Private Registry
The command to pull an image from a private registry is similar to pulling an image from Docker Hub. Here, first, we'll remove all the images with imageId 5d0da3dc9764:


We can see that images with imageId 5d0da3dc9764 have been removed. Let's check out the command to pull an image from the private Docker registry:

```
docker pull  localhost5000/baeldung-centos
```
### Set up Authentication for a Private Registry

Docker allows us to store the images locally on a centralized server, but sometimes, it's necessary to protect the images from external abuse. In that case, we'll need to authenticate the registry with the basic htpasswd authentication.

Let's first create a separate directory to store the Docker registry credentials:

```
 mkdir -p Docker_registry/auth
```
Next, let's run an httpd container to create a htpasswd protected user with a password:
```
 cd Docker_registry &&
docker run \
  --entrypoint htpasswd \
  httpd:2 -Bbn baeldung-user baeldung > auth/htpasswd
```

The above command will create a user with an htpasswd authenticated password. The details of the credentials are stored in the auth/htpasswd file.

Now, let's run the same Docker registry container using the auth/htpasswd authentication file:


```
docker run -itd \
  -p 5000:5000 \
  --name registry \
  -v "$(pwd)"/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry
  3a497bafed4adb21a5a3f0b52307b4beaa261c6abe265e543cd8f5a15358e29d
```
Since the Docker registry is running with the basic authentication, we can now test the login using:
```
$ docker login  localhost:5000 -u baeldung-user -p baeldung
```
Once successfully logged in to the Docker registry, we can both push and pull images in the same way we discussed above.
