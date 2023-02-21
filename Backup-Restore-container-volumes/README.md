## 1. Check the Existing Docker Volumes

```
docker volumes ls
```

## 2. Create the container on existing docker volume

```
docker run -it -v volume1:/containerdata --name container1 alpine
```
* volume1 = existing volume name
* containerdata = file name which create a directory on container
* container1 = container name
* alpine = image name

switch to new container and we coud see the containerdata folder on it.


## 3.Create backup on local 

```
docker run --rm --volumes-from container1 -v d:/volumebk:/backup ubuntu tar cvf /backup/backup.tar /containerdata
```

* volumebk = a directory on our locat drive d
* backup = create a new directoy on this conatiner


## 4 . Restore the backup.tar to another conatiner

### create a new conatiner

```
docker run -it -v /data --name restorecontainer ubuntu
```

### restore the file

```
docker run --rm --volumes-from restorecontainer -v d:/volumebk: /backup ubuntu bash -c "cd /data && tar xvf /backup/backup.tar"
```
