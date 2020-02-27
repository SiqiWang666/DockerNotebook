# Play with Container
- `docker run <image> [<override default command>]`: creating and running a container from an image. = `docker create <image>` + `docker start <containerID>`. 
    - `docker run -it <image> sh`: to start a shell instantly, it is a common use when *no primary process* need to be run first. `-i` attach terminal to STDIN, `-t` show up nicely formatted info, `sh` override default command.
    - Port Mapping: `-p <localHost port> : <container post>`
- `docker ps`: list all running containers (get ID), `--all`, list history containers (get ID for restart)
- `docker start -a <containerID>`: start a container, `-a` attatch STDOUT/STDEFF to this terminal. Note: when you resume a stopped container, you cann't overwrite the default command.
- `docker system prune`: clear up stopped containers and free up spaces
- `docker logs <containerID>`: get logs from a container
- `docker stop <containerID>`: SIGTERM -> running process, giving process some extra time to perform some backup operations. Actually, if the process don't terminate in 10 seconds, Docker will issue a kill command.
- `docker kill <containerID>`: shutdown immediately
- `docker exec -it <containerID> <command>` : run commands inside a container, `-i` attach terminal to STDIN, `-t` show up nicely formatted info
- `docker exec -it <contianerID> sh`: lauch a terminal from container itself instead of executing `docker exec` multiple times

# Build Image
## Dockerfile
`FROM alpine`, use base image of `alpine`. It seems like giving you a start point to proceed further work. `alpine` has some pre-installed handy programs you might need.
`RUN apk add --update redis`, `apk` package manager, 
`CMD ["redis-server"]`

## Build Process Under the Hood
`docker build .`: specify the dir of files/folders to use for the build
1. Step One: Download base image either from local cache or Docker public hub
1. Step Two
    - Create a container of previous step's image
    - Execute the command inside the container
    - Take snapshot of container's FS
    - Removing intermediate container
    - Return a temporary image.
1. Step Three
    - Create a container of previous step's image
    - Modify the container's primary startup command
    - Removing intermediate container
    - Return a successfully built image.

## Tag an Image
`docker build -t <tag> .`
Tag naming convention: `<DockerId>/<Repo/Project name>:<version>` When you run a container out of it, you don't necessarily specify version.

## Define Work Dir
`WORKDIR /usr/app`, specify the working directory of all later command

## Copy Files
`COPY ./ ./` the first `.`, the current working dir specified by `build` context argument. the second `.`, the current working dir of container `WORKDIR`

## Interesting
Create a container out of an image. Create an image out of a container. Haha!
`docker commit -c 'CMD ["redis-server"]' <containerID>`: `-c` specify the startup command

# Pulic Image on Docker Hub
The coventional keyword **alpine** represents the most compressed version.