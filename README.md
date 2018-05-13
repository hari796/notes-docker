# Notes => Docker

This repository was created to learn and practice docker concepts and tools

## Installing

My current distribution is Manjaro, it was really easy to install using the pacmac

## Docker daemon

I had some problems to get docker working, only the help command was working without the daemon running.
The daemon can be started with the command
```
sudo dockerd
```
After that, the docker commands worked but only with superuser permissions
```
sudo docker info
```

## Dockerfile => Image => Container

### Dockerfile

The Dockerfile is formed by a list of commands used to specify and configure the environment for the application that will run over.

```docker
# FROM command uses another Dockerfile as a basis for this, the python:2.7-slim can be found athttps://hub.docker.com/_/python/
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app
# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# EXPOSE makes the port number available outside the container
EXPOSE 80

# Define environment variable
ENV NAME World

# Component to run when the container launches
CMD ["python", "app.py"]
```

### Image

With the Dockerfile specification, an image can be created using the command
```
sudo docker build -t <image_name>
```
The image is the compiled Dockerfile specification, it already contains the application files (from Dockerfile directory) but it isn't running yet. The docker images can be listed with the command
```
sudo docker image ls
```

### Container

The container is the image running, to start it, I used the <image_name> set in the image creation, with the command
```
sudo docker run -d -p 4000:80 <image_name>
```
The -p argument maps a host port to the exposed port set in the Dockerfile

The -d argument is used to run the container detached from terminal, it still can be stopped with the command
```
sudo docker container stop <CONTAINER_ID>
```

The <CONTAINER_ID> can be found in the list of running containers using the command
```
sudo docker container ls (--all)
```
The --all (-a) argument will list a history of containers stopped or running

## Sharing the Image

The image can be shared in a Registry, that is a collection of Repositories, that is a collection of docker images.

Docker already provides a [cloud Registry](https://cloud.docker.com) but there are many other ones that can be used

Is possible to login using the docker CLI using the command
```
sudo docker login
```
It will ask for the account login and password

### Tag the Image

Is used to link a local image with a repository
```
sudo docker tag <image_name> <username>/<repository>:<tag>

Example:
sudo docker tag image pedro00dk/notes-docker:app
```
The [docker image ls]() will show the tagged images too

### Publish the Image

The app can be published with the command
```
docker push <username>/<repository>:<tag>

Example:
sudo docker tag image pedro00dk/notes-docker:app
```
When the command ends, the app will be available in the docker cloud

### Pull and run the image from the remote repository

```
docker run -p 4000:80 <username>/<repository>:<tag>

Example:
sudo docker run -p 4000:80 pedro00dk/notes-docker:app
```
If the published image isn't available locally, it will pull then run it
