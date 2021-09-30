#  Container lifetime and persistent data

## Persistent data handling

Containers are usually immutable  that means, only re-deploy containers, never change. Then what about unique or persistent data (like databases) ?

Docker solves this in two ways

- Data Volumes : makes special location outside of container Union File System (UFS)
- Bind Mount: This mounting host directory or file into the container. Link container path to host path. 

### Data Volumes

Data volumes is special location outside of container in host PC where the data resides. Actually Application in the container is made to write into a location within container, but this data is actually stored outside the container.

Advantage is even when we stop or remove container, the volume still exists and not deleted. We need to manually delete volume to remove it. This ensures data is not lost.  Thus volumes out live containers.

:octopus: Example from Dockerfile of mysql

```dockerfile
VOLUME /var/lib/mysql
```

On finding VOLUME in Dockerfile docker will create a unnamed volume with unique ID.

To create named Volume (user friendly) we need to use **-v** in command line and pass name of the volume

:octopus: Named Volume

```bash
docker container run -d --name <container-name> -v <volume-name>:<location-in-container> <image-name>
```

- **-v** : creates named volume. This can be new volume if not created, can be already existing volume.  

:octopus: List volume

```bash
docker volume ls
```

:octopus: Creating a volume

```bash
docker volume create <option>
```

### Bind Mounting

Maps a host file or directory to a container file or directory. Basically just two location (one from container side and other from host side) pointing to same file(s).

:bell: Using bind mount we can share file or directory between host and containers. Thus providing excellent facility during development and testing.

:bell: Bind mount can't be specified in **Dockerfile**. It can be given using command line during **container run**. 

:octopus: Bind mount of file directory using **-v** option during **docker run**

```bash
docker container run -d --name <container-name> -v <host-file-dir-location>:<container-file-dir-location> <image name>
```



