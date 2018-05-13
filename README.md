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