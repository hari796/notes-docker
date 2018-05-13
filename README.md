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

## Services

Services are 'containers in production'. A service manages how a single image runs, mapping the ports, replicating containers and more, in this way a service can control the number of containers running that image, assigning or reducing the computing power of this piece

### docker-compose.yml

This file controls the behavior of an image (published or local), setting how many containers will be used, mapping ports, limiting the memory and CPUs
```yml
version: "3"
# Services list (stack?)
services:
    # the service name (web)
    web:
        # pull the image from registry
        # image: <username>/<repository>:<tag>
        image: pedro00dk/notes-docker:app
        deploy:
            # Number of image replicas (generated containers)
            replicas: 5
            resources:
                limits:
                    # CPU usage ratio and memory per container replica
                    cpus: "0.1" # 0.1 => 10% of all cores
                    memory: 50M # 50M => 50 Megabytes
            restart_policy:
                condition: on-failure
        ports:
            # Maps the host ports to the service (web) ports
            - "80:80"
        networks:
            # Make the containers to share its ports (80) via webnet (it's a load balanced network), internally the containers publish to the service (web) 80 port using some random port
            - webnet
networks:
    # Sets webnet network with default configuration (which is a load-balanced overlay network)
    webnet:

```

## Run the load balanced app

The command [sudo swarn init]() shall be run a priori, or the subsequent commands will get errors

To start the service(s) the folowing command is used
```
sudo docker stack deploy -c docker-compose.yml <stack_name>

Example:
sudo docker stack deploy -c docker-compose.yml balancedapp
```
It creates and deploys a stack with all services in the docker-compose file, in this case, a stack with a single service with 5 containers running the same image and sharing the same port through a network middleware (webnet)

The services and stacks can be listed with the commands
```
sudo docker service ls
sudo docker stack ls
```
The name of a single container running in a service is **Task**, is possible to list the tasks of a service with the command
```
sudo docker service ps <stack_name>_<service_name>

Example:
sudo docker service ps balancedapp_web
```

## Stop the services (stack and swarm)

Stop the stack (app)
```
sudo docker stack rm <stack_name>

Example:
sudo docker stack rm balancedapp
```

Stop the swarm
```
docker swarm leave --force
```

## Swarm Clusters

Swarm is a group of machines running docker joined into a cluster, the docker commands can be executed on a cluster, they are managed by the **swarm manager**. The swarm machines can be physical or virtual, in a swarm, these machines are referred to as **nodes**, the swarm manager is one of these nodes too, and is the only that can execute the user commands, the other nodes are called **workers**, the workers have no authority and can be recruited by the swarm manager.

The swarm managers can use  some strategies to run containers (tasks) in the machines (nodes), these strategies can be set in the docker-compose file

### Set up the swarm

All the tasks except by the last (start the stack of services) were running in the **single-host mode**, but docker can also be switched to  **swarm mode**.
```
sudo docker swarm init
```
Enabling the use of swarm instantly makes the current machine a swarm manager. From now on, all commands will run on the swarm, and not only in the current machine

This command will provide the token, address and port used by workers to join the swarm

The command to add other machines as workers on the swarm is
```
sudo docker swarm join --token <token> <address>:<port>

Example:
sudo docker swarm join --token SWMTKN-1-3uyckzgn9ib8w4kl6hwnjgv7cs7f767prenexvijd3g71fq521-emp8z49leutyvxasxkici58il 192.168.1.11:2377
```

### Creating swarm cluster with VMs

With docker-machine it's possible to create VMs and connect then as nodes in a swarm.

The superuser privileges were not needed for docker-machine and for commands executed in the created vms

```
docker-machine create --driver <driver_name> <machine_name>

Example:
docker-machine create --driver virtualbox vm0
docker-machine create --driver virtualbox vm1
```
The <driver_name> is the type of the VM [backend](https://docs.docker.com/machine/drivers/).
The <machine_name> is the identifier for the VM

The machines are created and already set to start running, if they are stopped (reboot or logoff), they can be restarted with the command
```
docker-machine start <machine-name>

# If already running
docker-machine restart <machine-name>
```

The VMs can be listed with the command, it will show their IP addresses too, theses addresses will be used to connect the VMs to the swarm
```
docker-machine ls

Output:
NAME   ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
vm0    -        virtualbox   Running   tcp://192.168.99.100:2376           v18.05.0-ce
vm1    -        virtualbox   Running   tcp://192.168.99.101:2376           v18.05.0-ce
```
For more info about docker machines check enter [here](https://docs.docker.com/machine/overview/)

Is possible to run shell commands in the VMs using the following command
```
docker-machine ssh <machine_name> <command>

Example:
docker-machine ssh vm0 "uname -a"
Output:
Linux vm0 4.9.93-boot2docker #1 SMP Thu May 10 16:27:54 UTC 2018 x86_64 GNU/Linux

docker-machine ssh vm0 "docker swarm init --advertise-addr 192.168.99.100"
Output:
Swarm initialized: current node (re2s0tlpw4o146xhjw920xdna) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-20uubklicjyiax1w5g2kumf2ofzuapw02n8fl9iobpen5847ii-5mqzrru7khwm0v3dewkqdj4ms 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.


docker-machine ssh vm1 "docker swarm join --token SWMTKN-1-20uubklicjyiax1w5g2kumf2ofzuapw02n8fl9iobpen5847ii-5mqzrru7khwm0v3dewkqdj4ms 192.168.99.100:2377"
Output:
This node joined a swarm as a worker.
```
The created VMs already contains docker installed, so they can be configured as swarm managers or workers, configurations like the host machine as swarm manager and VMs as workers are possible too.
The configuration above has *vm0* as swarm manager and *vm1* as a worker

Is possible to add other swarm managers with the command
```
docker swarm <token> manager
```

Is possible to check the current nodes in the swarm cluster from the swarm manager using the command
```
docker node ls

Example:
docker-machine ssh <leader_vm> "docker node ls"
Output:
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
re2s0tlpw4o146xhjw920xdna *   vm0                 Ready               Active              Leader              18.05.0-ce
5axe8boj1zrzylty7odyykov7     vm1                 Ready               Active                                  18.05.0-ce
```
If this command is executed in a worker machine, it will end with error

## Deploy app on swarm cluster

To deploy the application on the swarm, is just repeat the commands used to deploy on the host machine for the swarm manager VM.

### Configure shell talk directly with VM (swarm manager)

The commands can be sent to the swarm manager VM by wrapping then with [docker-machine <machine_name> ssh command]() but another option is
```
docker-machine env <machine_name> # vm0
Output:
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/root/.docker/machine/machines/vm0"
export DOCKER_MACHINE_NAME="vm0"
# Run this command to configure your shell:
# eval $(docker-machine env vm0)

Run last line to make the shell talk to the vm
eval $(docker-machine env vm0)

sudo docker-machine ls
Output:
NAME   ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
vm0    *        virtualbox   Running   tcp://192.168.99.100:2376           v18.05.0-ce
vm1    -        virtualbox   Running   tcp://192.168.99.101:2376           v18.05.0-ce
```

To unset shell connection with VM run the command

```
 eval $(docker-machine env -u)
```

### Deploy app to swarm manager

With the shell talking directly with the active VM (swarm manager) is possible to easily execute commands in the VM (commands on VM don't need superuser privileges)
```
docker stack deploy -c docker-compose.yml <stack_name>

Example:
docker stack deploy -c docker-compose.yml stackbalancedapp
```

After that, the service address will not be the localhost, but any of the VMs addresses running the service, that can be obtained through [docker-machine ls]()

Is possible to check all containers (tasks) running on the machines (nodes) with the command
```
docker stack ps <stack_name>

Output:
ID                  NAME                     IMAGE                        NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
u82ne8hu33jf        stackbalancedapp_web.1   pedro00dk/notes-docker:app   vm0                 Running             Running 18 minutes ago                       
jowukynkbpcu        stackbalancedapp_web.2   pedro00dk/notes-docker:app   vm1                 Running             Running 18 minutes ago                       
xow7j2g8wd6g        stackbalancedapp_web.3   pedro00dk/notes-docker:app   vm1                 Running             Running 18 minutes ago                       
p6yc027u1q6d        stackbalancedapp_web.4   pedro00dk/notes-docker:app   vm0                 Running             Running 18 minutes ago                       
n28cbv4zvezm        stackbalancedapp_web.5   pedro00dk/notes-docker:app   vm1                 Running             Running 18 minutes ago   

```
After a [docker swarm join]() the command [docker stack deploy]() shall be called again, it will enable the new resources (nodes) for the app

## Stack

Is a group of services with shared dependencies that can be scaled together by a swarm manager

### Add new service and redeploy

New services can be added in the docker-compose file
```yml
    # The web service (already exists)
    web:
        ...
    # New visualizer service
    visualizer:
        image: dockersamples/visualizer:stable
        ports:
            - "8080:8080"
        # volumes is a new key that gives access to host socket file
        volumes:
            - "/var/run/docker.sock:/var/run/docker.sock"
        # deploy already exists in the web service but not the placement key
        deploy:
            # this key value ensures that this service only ever runs on the swarm manager node
            placement:
                constraints: [node.role == manager]
        networks:
            - webnet
```
To redeploy the stack, just run the stack deploy command again
```
docker stack deploy -c docker-compose.yml
```
This added service is an open-source project from Docker, used to display the services running in its swarm using a diagram such as [docker stack ps <stack_name>]()

### [Redis](https://redis.io/) (persisting the data)

Add a redis service in the docker-compose
```
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
        # like the visualizer service, it maps the host /home/docker/data path to the VM /data path 
      - "/home/docker/data:/data"
    deploy:
      placement:
        constraints: [node.role == manager]
    command: redis-server --appendonly yes
    networks:
      - webnet
```
Redis has an official image in the Docker library and has been granted the short image name of just redis, so no username/repo

The port 6379 has been pre-configured by Redis to be exposed from the container to the host, and it's exposed from host to world with the same number

Details about Redis image:
* redis always runs on the manager, so it’s always using the same filesystem
* redis accesses an arbitrary directory in the host’s file system as ```/data``` inside the container, which is where Redis stores data

The ```/data``` foldar folder needs to be created in the swarm manager VM with the command
```
docker-machine ssh <leader_vm> "mkdir ./data"
# <leader_vm> is the swarm manager vm, in this case, vm0
```
After that, redeploy the application
```
docker stack deploy -c docker-compose.yml
```
