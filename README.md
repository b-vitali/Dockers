# My Dockers
This is just a handy collection of dockerfiles to create some dockers or useful dockerhubs

## Table of content
- [Setup](#setup)
- [pyTorch](#pytorch)
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

</details>

### Plot forwarding

If you want to have plots and graphs you need to have the proper forwarding.

> [!IMPORTANT] Linux
> On your side run ```xhost +local``` and remember ```--volume /tmp/.X11-unix:/tmp/.X11-unix```

> [!IMPORTANT] Windows
> Yyou will need VcXsrv or Xming 
> Run it creating `multi-windows` and set the `display number to 0`

### Shortcuts

You will probably have a long command to start the docker in the propre place with all the flags on. 
We can make it simpler creating an 'alias' for the command

> [!IMPORTANT] Linux
> In your `.bashrc` create an `alias`
> ```
> alias run_pytorch='docker run -it --rm --name pytorch -p 8888:8888 --env DISPLAY=$DISPLAY --volume /tmp/.X11-unix:/tmp/.X11-unix --volume local/path/to/files:/mnt/pytorch pytorch-docker' 
> ```
> This will create an `alias` you can run in the `terminal`

> [!IMPORTANT] Windows
> If you use `powershell` you can create a funcion in your `$PROFILE`, something like
> ```
> function run_pytorch {
>   docker run -it --rm --name pytorch -p 8888:8888 --env DISPLAY=host.docker.internal:0 --volume local\path\to\files:/mnt/pytorch pytorch-docker
> }
> ```
> This will create a function you can run in the powershell
> NB: you need to add the `$PROFILE` to the `$PATH`

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


## pyTorch
As described on the [website](https://pytorch.org/), *PyTorch is an optimized tensor library for deep learning using GPUs and CPUs*. It provides a flexible and dynamic computational graph, making it easy to build, train, and deploy machine learning models. 
PyTorch is widely used for tasks such as computer vision, natural language processing, and reinforcement learning.

<details>
<summary>Build and use</summary>
Standard build specifying the dockerfile name

```docker build -t pytorch-docker -f pytorch_Dockerfile . ```


```
docker run -it --rm \
    --name pytorch \
    -p 8888:8888 \
    --env DISPLAY=$DISPLAY \
    --volume /tmp/.X11-unix:/tmp/.X11-unix \
    --volume /home/bastiano/Software/pytorch-docker:/mnt/pytorch \
    pytorch-docker 
```

on Windows is siglty different
```
    docker run -it --rm --name pytorch -p 8888:8888 --env DISPLAY=host.docker.internal:0 --volume C:\Users\bastiano\Documents\ML_curveprediction:/mnt/pytorch pytorch-docker
```

</details>

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


In the last iteration i added `screen` and PS1 for them:
```
# How do you want to call the project
RUN echo 'export CUSTOM_NAME="x-suite"' >> /home/user/.bashrc

# Set the terminal title and PS1 with screen
RUN echo 'echo -ne "\033]0;X-Suite\007"' >> /home/user/.bashrc
RUN echo 'if [[ $STY ]]; then export PS1="\[\033[01;32m\]${CUSTOM_NAME}_$STY\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ "; else export PS1="\[\033[01;32m\]${CUSTOM_NAME}\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ "; fi' >> /home/user/.bashrc

# Create .screenrc file and set bash as the shell for screen
RUN echo "shell /bin/bash" >> /home/user/.screenrc
RUN echo "source /home/user/.bashrc" >> /home/user/.screenrc
```

This is a bit confusing but the idea is to have a clean terminal.
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

If you want to make a `ConvertExample.html` you can run

`manim-slide render ConvertExample.py` and then 

`manim-slide converter ConvertExample ConvertExample.html`

This will create a presentation in html format to be open in most browsers

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
