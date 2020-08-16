# Play with Container
- `docker run <image> [<override default command>]`: creating and running a container from an image. = `docker create <image>` + `docker start <containerID>`.
    - `docker run -it <image> sh`: to start a shell instantly, it is a common use when *no primary process* need to be run first. `-i` attach terminal to STDIN, `-t` show up nicely formatted info, `sh` override default command.
    - **Port Mapping**: `-p <localHost port> : <container post>`
    - **Volume**: Map folder inside a container to folder outside a container. `-v $(pwd):/app`: Map the `pwd` into the `/app` foler. `-v /app/node_module`: Put the bookmark on the folder, i.e., do not map this folder.
- `docker ps`: list all **running** containers (get ID), `--all`, list history containers (get ID for restart)
- `docker start -a <containerID>`: start a container, `-a` attatch STDOUT/STDERR to this terminal. Note: when you resume a stopped container, you cann't overwrite the default command.
- `docker system prune`: clear up stopped containers and free up spaces
- `docker logs <containerID>`: get logs from a container
- `docker stop <containerID>`: SIGTERM -> running process, giving process some extra time to perform some backup operations. Actually, if the process don't terminate in 10 seconds, Docker will issue a kill command.
- `docker kill <containerID>`: shutdown immediately
- `docker exec -it <containerID> <command>` : run commands inside a container, `-i` attach terminal to STDIN, `-t` show up nicely formatted info
- `docker exec -it <contianerID> sh`: lauch a terminal from container itself instead of executing `docker exec` multiple times

# Build Image

## Dockerfile

### Base Image
`FROM alpine`, use base image of `alpine`. It seems like giving you a start point to proceed further work. `alpine` has some pre-installed handy programs you might need.
> The coventional keyword **alpine** represents the most compressed version.

### Define Work Dir
`WORKDIR /usr/app`, specify the working directory of the container. So all later command will be executed under working directory.

### Install Dependencies
`RUN apk add --update redis`, `apk` package manager.

### Copy Files
`COPY ./ ./` The first `.`, the current working dir specified by `build` context argument. The second `.`, the current working dir of container `WORKDIR`.

```
Local Folder
  .
  |- src
  |_ pulic

Docker Container
  .
  |- src
  |_ public
```
### Startup Command
`CMD ["redis-server"]`, exec form

## Dockerfile.dev
A Dockerfile for development.
Note: when you build image out of `Dockerfile.dev` make sure you add file option `docker build -f Dockerfile.dev .`, because `docker build` would automatically search for `Dockerfile`.

## Build Process Under the Hood
`docker build .`: specify the dir of files/folders to use for the build

> Using option `--rm=true` to remove intermediate containers

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

## Multi-step Builds
For example, you are building a static React server. We use image `Nginx` to host static assets.

```bash
FROM node:alpine as builder
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
Run npm run build

FROM nginx
# Copy from previous step
COPY --from=builder /app/build /usr/share/nginx/html
```

## Tag an Image
`docker build -t <tag> .`
Tag naming convention: `<DockerId>/<Repo/Project name>:<version>` When you run a container out of it, you don't necessarily specify version.


## Interesting
Create a container out of an image. Create an image out of a container. Haha!
`docker commit -c 'CMD ["redis-server"]' <containerID>`, `-c` specify the startup command

# Docker Compose
Docker Compose funtions like Docker CLI, but allows you to issue commands much more quickly.
`docker-compose.yml` allows you to create multiple containers in a single file. In addition, `docker-compose` will automatically set up communications (networking) between those containers.
```bash
# specify the docker-compose formatting version
# '-' means array in .yml
version: '3'
# list all the available services
services:
  # Container running redis-server
  # Traditionally application has a database driver URL to connect, here the URL is `redis-server`
  redis-server:
    # specif image
    image: 'redis'
  # Container running node-app
  node-app:
    # Build image out of current dir, Dockerfile.
    build: .
    # Build image NOT out of current dir and NOT conventional dockerfile
    # build:
    #   context: /app
    #   dockerfile: Dockerfile.dev

    # Map localPort : containerPort
    ports:
      - "4001:8081"
    volumes:
      # Don't try to map up against /app/node_module
      - /app/node_module
      # Every time container access `/app` folder, it will redirect to . 
      - .:/app
    # specify environment variables
    environment:
      - REDIS_HOST=redis-server
      - REDIS_POST=6379
    # Specify restart policy for node-app, check out restart policy cheatsheet
    restart: always

  # Start second service responsible for testing
  tests:
    build: .
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm", "run", "test"]
```
- `docker-compose up`: start containers
- `docker-compose up --build`: rebuild and start containers
- `docker-compose down`: stop containers
- `docker-compose ps`: list the containers inside `docker-compose.yml` file

# CI/CD Workflow
continuous integration + continous delivery/deployment
## Travis CI

### Link GitHub to Travis CI
Head to dashboard of Travis, then search your repo and link it. (If you don't have an account, sign up one).

### Travis CI Confinuration
Configuration of Travis CI: `.travis.yml`
```bash
# Have super-user level permission
sudo: required
# Tell Travis CI we need docker CLI pre-installed
services: 
  - docker

# Commands need to be executed before testing
before_install:
  # Build image, return image ID, using development dockerfile
  - docker build -t dockerUserName/appName -f Dockerfile.dev .

# Script for test being executed
script:
  - docker run <imageID> npm run test -- --coverage

after_success:
  - docker build -t dockerUserName/appName .
  # Log in to the docker CLI
  - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_OD" --password-stdin
  # Push image to docker hub
  - docker push dockerUserName/appName
# Script for deployment
deploy:
  # Tell Travis CI what platform I am using
  provider: elasticbeanstalk
  # info can be found from env URL on elastic beanstalk dashboard.
  region: "us-west-2"
  app: "docker"
  env: "Docker-env"
  # info can be found from S3
  bucket_name: 
  bucket_path:
  # Only deploy when changes are made on master branch
  on:
    branch: master
  # Create a new user for Travis CI to access service on IAM. Return Access key ID and secret key.
  # Add those environment variable to Travis CI.
  access_key_id: $AWS_ACCESS_KEY
  secret_access_key:
    secure: "$AWS_SECRET_KEY"
```
If the exit code is `0`, everything works successfully.

## AWS Elastic Beanstalk
Create a new application. Then create an environmnet, docker.

# Multi-containers Deployment
React, Express, Redis, PostgreSQL

## Help from Nginx

Nginx can be used as router, static assets host, load balancer, etc.

### Static Files Host
First, we need to create `default.conf` and then overwrite it so that nginx will perform as expected.
```bash
server {
  listen 3000;
  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;
  }
}
```

Then we need to create production level Dockerfile (like multi-step build).
```bash
FROM node:alpine as builder
WORKDIR '/app'
COPY ./package.json .
RUN npm install
COPY . .
RUN npm run build

FROM nginx
EXPOSE 3000
# overwrite config files
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
# copy over static files from previous step
COPY --from=builder /app/build /usr/shar/nginx/html
```

# Things to Keep in Mind

Slimming down your docker image.

- small images are faster to copy around,
- they are faster to start,
- by removing unneeded things you are reducing the damage that a security exploit could do.

# Kubernete

A **cluster** consists of several nodes and a master. A **node** is either a VM or physical computer. There are several containers running on a node. **Master** controls which node does.


*minikube*: use for managing the VM itself (LOCAL ONLY) `brew install minikube`. *kubectl*: use for managing containers in the node (ctl for interacting with master). `brew install kubectl`

## How to start
`minikube start`, automatically setup a VM (virtual box installed) and configure it to behave as a node in the cluster.