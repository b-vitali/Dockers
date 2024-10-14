# My Dockers
This is just a handy collection of dockerfiles to create some dockers or useful dockerhubs

## Table of content
- [Setup](#setup)
- [X-Suite](#x-suite)
- [CERN ROOT](#cern-root)
- [Manim+ManimSlides](Manim+ManimSlides)
- [G4bl and GEANT4 (Coming Soon)](#g4bl-and-geant4-coming-soon)

## Setup
This is **not** a guide, just some note for my future reference.


<details>
<summary>If you already have a running docker just skip this... otherwise CLICK!</summary>

### Preparations

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

### Install
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

</details>


### Useful Commands
You can find all the usual stuff running `docker -h`, but here are some important things:
> [!TIP]
>
> - Info on the memory usage of the different images/containers: `docker system df`
> - Info on the images: `docker images -a`
> - Info on the containers: `docker container ls`
> - To remove stuff there are:
>   - `docker system prune`
>   - `docker rmi <imageID>`
>   - `docker rm <containerID>`


## X-Suite
As described on [x-suite website](https://xsuite.readthedocs.io/en/latest/):
*Xsuite is a collection python packages for the simulation of the beam dynamics in particle accelerators.*

This docker has some minimal tweeking to have a functioning jupyter notebook and other small things.
Most of the requirements for additional tools (MAD-X, Sixtracktools, PyHEADTAIL, ...) are already installed.


<details>
<summary>Build and use</summary>

Standard build specifying the dockerfile name
```docker build -t xsuite-docker -f x-suite_Dockerfile . ```

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

If you want to run X-Suite via python you just import it and run `python3 MyFile.py`

To run jupyter just run ``` myjupyter ``` then open your browser and go to ```http://127.0.0.1:8888/tree/mnt/x-suite```

</details>


## Manim + ManimSlides
Manim is an animation engine for explanatory math videos. 
It's used to create precise animations programmatically, as demonstrated in the videos of [3Blue1Brown](https://www.3blue1brown.com/).
My docker is based on the version maintained by the [Manim Community](https://github.com/ManimCommunity), which is not associated with 3Blue1Brown.

This Docker is not small because it comes with python, manim, manim slides and Latex.

Is everything necessary? I don't know yet and I will update it when I will figure it out!


<details>
<summary>Build and use</summary>

Standard build specifying the dockerfile name
```docker build -t manim-docker -f manim_Dockerfile . ```

To then run the image is always the same stuff:
```
docker run -it --rm \
    --name manim \
    -p 8888:8888 \
    --env DISPLAY=$DISPLAY \
    --volume /tmp/.X11-unix:/tmp/.X11-unix \
    --volume /home/bastiano/Software/manim-docker:/mnt/manim \
    manim-docker
``` 

To use this python package you call `manim [OPTIONS] MyFile.py`

If you want to make a `Slides.html` you can run
`manim-slide render MyFile.py Presentation` and then `manim-slide converter Presentation Slides.html`

</details>

## CERN ROOT
Instead of creating it from scratch, root can be downloaded directly

<details>
<summary>Pull and run</summary>

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
</details>


### G4bl and GEANT4 (Coming Soon)
