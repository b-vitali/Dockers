# My Dockerfile(s)
This is just a handy collection of dockerfiles to create some dockers or useful dockerhubs

1. [Setup](#setup)
    - [X11](#x11)
    - [Useful Commands](#useful-commands)
4. [X-Suite](#x-suite)
    - [Build and Use](#build-and-use)
5. [CERN ROOT](#cern-root)
    - [Pull and Run](#pull-and-run)
6. [G4bl and GEANT4 (Coming Soon)](#g4bl-and-geant4-coming-soon)


## Setup
This is **not** a guide, just some note for my future reference.

Make sure you are up to date and add the necessary tools
```
sudo apt update 
sudo apt upgrade 
sudo apt install apt-transport-https ca-certificates curl software-properties-common 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - 
echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list 
```

Last time, I had to fix the version. Default was wilma I needed jammy 
```
sudo apt update 
sudo nano /etc/apt/sources.list.d/docker.list  
```
 
I'm on Linux Mint... Ubuntu has deprecated storing GPG keys in the legacy /etc/apt/trusted.gpg keyring.  
Instead, keys should be stored in individual files under /etc/apt/trusted.gpg.d/  
So: find docker key -> remove it -> make new -> edit with new info the docker.list 
```
sudo apt update 
apt-key list 
sudo apt-key del 0EBFCD88 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg 
sudo nano /etc/apt/sources.list.d/docker.list 
deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu jammy stable 
```

Finally I can install and check if it runs 
```
sudo apt update 
sudo apt install docker-ce 
docker -v 
sudo docker run hello-world 
```


### X11
If you want to have plots and graphs you need to have X11 forwarding.
On your side run ```xhost +local``` and remember ```--volume /tmp/.X11-unix:/tmp/.X11-unix``` when running docker

### Useful commands
You can find all the usual stuff running ```docker -h``` but here are some important thigs:

Info on the memory usage of the different images/containers ```docker system df```
Info on the images ```docker images -a ```
Info on the containers ```docker container```  
To remove stuff there are: ```docker prune``` ```docker rmi <imageID>``` ```docker rm  <containerID>``` 


## X-Suite
As described on [x-suite website](https://xsuite.readthedocs.io/en/latest/):
*Xsuite is a collection python packages for the simulation of the beam dynamics in particle accelerators.*

This docker has some minimal tweeking to have a functioning jupyter notebook and other small things.
Most of the requirements for additional tools (MAD-X, Sixtracktools, PyHEADTAIL, ...) are already installed.

### Build and use
Standard build specifying the dockerfile name
```docker build -t Dockerfile_x-suite xsuite-docker .```

To run it and having your browser access *jupyter* and the popup *plots* we have some flag to add
```
docker run -it --rm \
    --name xsuite \
    -p 8888:8888 \
    --env DISPLAY=$DISPLAY \
    --volume /tmp/.X11-unix:/tmp/.X11-unix \
    --volume /home/bastiano/Software/x-suite-docker:/mnt/x-suite \
    xsuite-docker
``` 

To run jupyter just run ``` myjupyter ``` then open your browser and go to ```http://127.0.0.1:8888/tree/mnt/x-suite```

## CERN ROOT
Instead of creating it from scratch, root can be downloaded directly

### Pull and run
Just like a git repo, simply *pull* the latest version
```docker pull rootproject/root```

To run the image, connect the display, makes so you can access the files run this command
NB ```--user $(id -u):$(id -g)``` makes sure the files you create are "yours"
```
docker run –it --rm \
    --env DISPLAY=$DISPLAY \
    --volume /tmp/.X11-unix:/tmp/.X11-unix \
    --volume /home/bastiano/Documents:/mnt/Documents \
    --user $(id -u):$(id -g)   
    rootproject/root /bin/bash 
```

### G4bl and GEANT4 (Coming Soon)