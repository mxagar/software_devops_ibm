# IBM DevOps and Software Engineering Professional Certificate: Introduction to Containers w/ Docker, Kubernetes & OpenShift

These are my notes of the course [Introduction to Containers w/ Docker, Kubernetes & OpenShift](https://www.coursera.org/learn/ibm-containers-docker-kubernetes-openshift?specialization=devops-and-software-engineering).

I have some related repositories in which I compile a guide on Docker; this course is suposed to focus on Kubernetes.

- [udemy-docker-mastery](https://github.com/mxagar/udemy-docker-mastery)
- [tool_guides/docker_swarm_kubernetes](https://github.com/mxagar/tool_guides/tree/master/docker_swarm_kubernetes)

Table of contents:

- [IBM DevOps and Software Engineering Professional Certificate: Introduction to Containers w/ Docker, Kubernetes \& OpenShift](#ibm-devops-and-software-engineering-professional-certificate-introduction-to-containers-w-docker-kubernetes--openshift)
  - [1. Introduction: Docker Containers](#1-introduction-docker-containers)
    - [1.1 First Example: Build, Run, Pull, Push](#11-first-example-build-run-pull-push)
    - [1.2 Docker objects](#12-docker-objects)
    - [1.3 Docker Architecture](#13-docker-architecture)
    - [1.4 Exercises](#14-exercises)
      - [First interaction](#first-interaction)
      - [Building the image](#building-the-image)
      - [Running the image](#running-the-image)
      - [Push image to IBM Cloud Registry](#push-image-to-ibm-cloud-registry)
      - [Cheatsheet](#cheatsheet)
  - [2. Setup and Installation](#2-setup-and-installation)
    - [Installation Docker on WSL](#installation-docker-on-wsl)
    - [Creating an IBM Cloud Account](#creating-an-ibm-cloud-account)
  - [3. Kubernetes Basics](#3-kubernetes-basics)

## 1. Introduction: Docker Containers

Benefits of containers:

- Apps and microservices encapsulated.
- Containers are platform independent.
- Scaling possible.
- Microservice applications become much easier.
- etc.

Docker / Containers are not the best option if:

- We need high performance
- The architecture is monolithic
- We have rich GUI features
- The app is a very limited desktop app

Concepts:

- Image
- Container: runnable instance of an image
- Dockerfile: image specification file
- Namespace: isolated workspace, where containers run
- Registries

### 1.1 First Example: Build, Run, Pull, Push

`Dockerfile`:

```Dockerfile
# Base image
FROM alpine
# Command
CMD ["echo", "Hello World!"]
```

Building and running:

```bash
# Build
# -t: tag
# my-app: repository
# v1: version
docker build -t my-app:v1

# Get all images
docker images

# Create/Run container
docker run my-app:v1

# Get running containers
docker ps -a

# Push images to a registr we're logged in, i.e., configured
docker push my-app:v1

# Retrive images from configured registries
docker pull nginx
```

### 1.2 Docker objects

Dockerfile:

```Dockerfile
# Common commads
FROM # base image
RUN # execute commands
CMD # default command for execution
```

Docker images:

- Instructions to build a container
- Composed by layers added sequentuially; when image is changed, only the layers on the top of the change are modified.

Image name:

```
hostname/repository:tag
docker.io/ubuntu:18.04
```

Other objects:

- Networks are used for isolated container communications
- Storage: volumes are used for persisting data after container stops
- Plugins: connect to external platforms

### 1.3 Docker Architecture

- Interaction via CLI, REST API or GUI
- Docker host server has the docker daemon: `dockerd`
  - It is distirbuted: the host server and the execution don't need to be on the same physical machine
- Daemon listens to interactions
- Daemon executes the commands
- Registry stores images: public, private

![Docker Architecture](./pics/docker_architecture.jpg)

### 1.4 Exercises

An environment is generated with a Terminal.

The working directory contains a simple Node.js application that we will run in a container. The app will print a hello message along with the hostname. The following files are needed to run the app in a container:

- `app.js` is the main application, which simply replies with a hello world message.
- `package.json` defines the dependencies of the application.
- `Dockerfile` defines the instructions Docker uses to build the image.

I have replicated the folder in [`lab/01_ContainersAndDocker/`](./lab/01_ContainersAndDocker/), so the exercise can be carried out locally.

#### First interaction

```bash
docker --version
ibmcloud version
cd /home/project
# Clone a repository
[ ! -d 'CC201' ] && git clone https://github.com/ibm-developer-skills-network/CC201.git
cd CC201/labs/1_ContainersAndDocker/
ls # app.js  Dockerfile  package.json
docker pull hello-world
docker images # now, image is there
docker run hello-world
docker ps -a # containers with IDs
docker container rm <container_id> # f36c514edcd4
docker ps -a # container not there anymore
```

#### Building the image

```Dockerfile
FROM node:9.4.0-alpine
COPY app.js .
COPY package.json .
RUN npm install &&\
    apk update &&\
    apk upgrade
EXPOSE  8080
CMD node app.js
```

```bash
# Build the image
docker build . -t myimage:v1
# Check image is there
docker images # base images appear, too
```

#### Running the image

```bash
docker run -dp 8080:8080 myimage:v1
curl localhost:8080
# Hello world from 0c0a10807bfd! Your app is up and running!

# Stop all running containers
docker stop $(docker ps -q)
# Check nothing is running
docker ps
```

#### Push image to IBM Cloud Registry

We have a custom (automatically generated IBM Cloud account).

```bash
# Check we're logged in
ibmcloud target

# Check namespaces we have access to
ibmcloud cr namespaces

# Set namespace region
ibmcloud cr region-set us-south

# Log local docker to IBM Cloud Container Registry to pull/push
ibmcloud cr login

# Create namespace env variable
export MY_NAMESPACE=sn-labs-$USERNAME

# Tag image befoe pushing
docker tag myimage:v1 us.icr.io/$MY_NAMESPACE/hello-world:1

# Push to registry the tagged image
docker push us.icr.io/$MY_NAMESPACE/hello-world:1

# Check the image is in the registry
ibmcloud cr images

# To view only images in my namespace
ibmcloud cr images --restrict $MY_NAMESPACE
```

#### Cheatsheet

Also, check my summary: [`docker_commands_summary.txt`](./docker_commands_summary.txt).

```bash
curl localhost 	# Pings the application.
docker build 	# Builds an image from a Dockerfile.
docker build . -t 	# Builds the image and tags the image id.
docker CLI 	# Start the Docker command line interface.
docker container rm # Removes a container.
docker images 	# Lists the images.
docker ps 	# Lists the containers.
docker ps -a 	# Lists the containers that ran and exited successfully.
docker pull 	# Pulls the latest image or repository from a registry.
docker push 	# Pushes an image or a repository to a registry.
docker run 	# Runs a command in a new container.
docker run -p 	# Runs the container by publishing the ports.
docker stop 	# Stops one or more running containers.
docker stop $(docker ps -q) 	# Stops all running containers.
docker tag 	# Creates a tag for a target image that refers to a source image.
docker –version 	# Displays the version of the Docker CLI.
exit 	# Closes the terminal session.
export MY_NAMESPACE 	vExports a namespace as an environment variable.
git clone 	# Clones the git repository that contains the artifacts needed.
ibmcloud cr images 	# Lists images in the IBM Cloud Container Registry.
ibmcloud cr login 	# Logs your local Docker daemon into IBM Cloud Container Registry.
ibmcloud cr namespaces 	# Views the namespaces you have access to.
ibmcloud cr region-set 	# Ensures that you are targeting the region appropriate to your cloud account.
ibmcloud target 	# Provides information about the account you’re targeting.
ibmcloud version 	# Displays the version of the IBM Cloud CLI.
ls 	# Lists the contents of this directory to see the artifacts. 
```

## 2. Setup and Installation

Most of this section was done by myself, it's not included in the course.

Check: [tool_guides/docker_swarm_kubernetes](https://github.com/mxagar/tool_guides/tree/master/docker_swarm_kubernetes).

### Installation Docker on WSL

- []()

### Creating an IBM Cloud Account

Check my notes at []().

Also, there might be a symbolic link in the local cloned folder: []().

## 3. Kubernetes Basics

:construction:

TBD.
