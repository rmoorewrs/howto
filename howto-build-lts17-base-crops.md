# WR Linux LTS17 BASE Platform build using Mirror, CROPS/Poky and Docker

CROPS (CROssPlatformS) is a Yocto project aimed at providing a consistent build environment and avoiding build host contamination through the use of Linux containers. Docker is used in this example.

The build and mirror directory live on your build host and are temporarily mounted by your CROPS container. Many options are possible but the defaults here will create the CROPS container on demand and delete it on exit. The contents of the mirror and build directory are always left intact.

>Note: 
- You can successfully build without the mirror, by cloning your final project directly from github. However, a mirror will reduce the number of do_fetch() errors you might encounter on any given day and is recommended.
- Depending on your host OS, the Crops container is often not necessary. Again, Crops gives a consistent build environment and reduces the possibility of build host-related errors, once set up, its very easy to use ... but optional.


## Requirements for build host/VM:
- A host (Linux, Mac, Windows) that supports a recent version of docker
- see the docker website and/or your distro's instructions for installing docker.
- sudo privilege
- external internet access
- python 2.7.x (python 3 may work as well)
- about 100GB of free space for the WRL LTS17 OVP build, less for other images
- Host packages (for Ubuntu):
```
sudo apt update
sudo apt install gawk wget git-core diffstat unzip texinfo gcc-multilib \
     build-essential chrpath socat cpio python python3 python3-pip python3-pexpect \
     xz-utils debianutils iputils-ping libsdl1.2-dev xterm ssh-askpass docker.io
```

### 1) Install and test docker on your build machine/VM
Log into a bash shell on your build machine/VM. Install docker community edition per your instructions for your platform on the docker site:
```
https://docs.docker.com/v17.09/engine/installation/
```

Add your user to the docker group. You must log out and back in again the first time for the group membership to take effect. This allows you to run docker without 'sudo'. Use your actual login in place of 'my_userid'.
```
$ sudo usermod -aG docker <my_userid>
```
Run two quick tests: 

Check docker version
```
$ docker version
wruser@wrl-build:~$ docker version
Client:
 Version:      1.13.1
 API version:  1.26
 ...
```

Try the `hello-world` container:
```
$ docker container run --rm hello-world
```
>Note: if you get an error, either the docker service isn't running or you failed to add yourself to the correct docker group.

### 2) Create your local mirror repo
Wind River Linux 9 and LTS17 builds get packages from the internet by default. Inevitably, a random server will be down, causing bitbake `do_fetch` fails. These usually come back after hours or days (and after you've wasted time filing JIRA bugs).

The best practice to is to use a local mirror repo as the source for your builds. 
>Note: the mirror looks like a regular project, but you don't typically build anything in it.

To create your mirror, first create a directory:
```
$ cd <my_wrl_path>
$ mkdir mirror
$ cd mirror
```
Clone from the github repo like any other build:
```
$ git clone --branch WRLINUX_10_17_BASE https://github.com/WindRiver-Labs/wrlinux-x.git
```

Next, configure the mirror with setup.sh (this is where the project becomes a mirror):
```
$ ./wrlinux-x/setup.sh --mirror --all-layers --dl-layers
```
> Note: in order to update later on, cd into the mirror's wrlinux-x directory and do a `git pull` followed by running the above setup line again.

### 3) Clone and setup your LTS17 project
Create a directory on the build host where for your actual platform project. 
```
$ mkdir mybuild
$ cd mybuild
```

Here's how to setup a build for glibc-core image with the intel-corei7-64 BSP. Substitute your mirror path below for $MIRROR:
```
$ git clone --branch WRLINUX_10_17_BASE $MIRROR/wrlinux-x  
$ ./wrlinux-x/setup.sh \
--machines intel-corei7-64 \
--distros wrlinux \
--dl-layers
```

After running setup, you need to edit the build/conf/local.conf file to make the following changes (along with any others you wish to make):
1. Enable network access
```
BB_NO_NETWORK ?= '1'
```
to
```
BB_NO_NETWORK ?= '0'
```

2. Uncomment the line that specifies the virtual/kernel:
```
PREFERRED_PROVIDER_virtual/kernel = "linux-yocto"
```
to
```
PREFERRED_PROVIDER_virtual/kernel = "linux-yocto"
```

3. Add a line to whitelist some Intel packages:
```
PNWHITELIST_intel += 'rmc-efi rmc thermald intel-microcode iucode-tool-native'
```

### 4) Start the crops/poky container (optional, but recommended)
```
$ docker container run --rm -it -v $(pwd):$(pwd) crops/poky --workdir=$(pwd)
```
Note: The first time this runs, it will take a few minutes to download the initial crops image. After that, it will start very quickly.

Now your bash prompt will be inside the container and the prompt will look something like this:
```
pokyuser@bfd43fd1cd9b:/home/builduser/mybuild$
```


### 4) Build the LTS17 platform:
While still in the crops container:
```
$ . ./environment-setup-x86_64-wrlinuxsdk-linux
$ . ./oe-init-build-env
```
And then run bitbake for the glibc-core image:
```
$ bitbake wrlinux-image-glibc-core
```

This will take awhile...

>Note: when you finish and exit the container, the project will be available right where you created it before starting the container.

### TIP: Create a 'runcrops' alias 
To make life easier, create a convenience alias called 'runcrops' so you don't have to remember the docker command line. A simple example would be to add the following lines to the .bashrc file on your build machine:
```
alias runcrops='docker container run --rm -it -v $(pwd):$(pwd) crops/poky --workdir=$(pwd)'
```
After your .bashrc is sourced, you should be able to cd into your project directory and run `runcrops`
```
$ cd mybuild
$ runcrops
pokyuser@7f7430d76fb6:/home/wruser/mybuild$
``` 
