# Docker Client
- `docker run <image> [<override default command>]`: creating and running a container from an image. = `docker create <image>` + `docker start <containerID>`. `-it`: start a shell instantly, it is common when *no primary process* need to be run first.
- `docker ps`: list all running containers, `--all`, list history containers
- `docker start -a <containerID>`: start a container, `-a` specifies output info on this terminal. Note: when you resume a stopped continer, you cann't overwrite the default command.
- `docker system prune`: clear up stopped containers and free up spaces
- `docker logs <containerID>`: get logs from a container
- `docker stop <containerID>`: SIGTERM -> running process, giving process some extra time to perform some backup operations. Actually, if the process don't terminate in 10 seconds, Docker will issue a kill command.
- `docker kill <containerID>`: shutdown immediately
- `docker exec -it <containerID> <command>` : run commands inside a container, `-i` attach terminal to STDIN, `-t` show up nicely formatted info
- `docker exec -it <contianerID> sh`: lauch a terminal from container itself instead of `docker exec` multiple times