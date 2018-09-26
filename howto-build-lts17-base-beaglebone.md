## How to Build a Beaglebone Image with WRL LTS17 Base (Community Edition)
This note shows how to build a WR Linux LTS17 image for the Beaglebone board using the Wind River Linux community edition. The Community Edition of Wind River Linux LTS is located on github here: 
```
https://github.com/WindRiver-Labs/wrlinux-x/tree/WRLINUX_10_17_BASE
```
Detailed Documentation is available from the Wind River Knowledge Library here:
```
https://knowledge.windriver.com/en-us/000_Products/000/010/060
```
>Note: Since the Yocto pulls from many online repositories, often one of the repos will be down temporarily causing intermittent build problems (failures in `do_fetch`). Therefore,  it's best practice to create a local mirror repository for more reliable builds.

### Requirements
- 64-bit Linux host with Python 3 and git version 1.9 or higher.
- internet access
-  200GB+ of free disk space (some images require more space)
See `https://knowledge.windriver.com/en-us/000_Products/000/010/060/000/000/000_Wind_River_Linux_Release_Notes%2C_LTS_(RCPL_11)/020` for a list of supported hosts.

### Step 1: Clone the WRL LTS17 Base Repo to create a local mirror
Create a mirror directory, clone repo from github and set up the mirror:
> Note: never do any operations here as `root` or with `sudo` unless specifically directed to do so.
```
$ mkdir mirror
$ cd mirror
$ git clone https://github.com/WindRiver-Labs/wrlinux-x.git
$ wrlinux/setup.sh --all-layers --mirror -dl-layers
```
See `https://knowledge.windriver.com/en-us/000_Products/000/010/060/000/000/000_Wind_River_Linux_Getting_Started%2C_LTS/050/010` for more details

### Step 2: Create a Platform build directory for the Beaglebone WRL Image
Create a build directory for the beaglebone image **anywhere outside of the mirror directory**. Clone from the local mirror you just set up and configure the project setup:
```
$ mkdir beaglebone
$ cd beaglebone
$ git clone --branch WRLINUX_10_17_BASE <path-to-mirror>/mirror/wrlinux-x
$ wrlinux/setup.sh --machines beaglebone --distros wrlinux --dl-layers
```
There a couple of files worth reading in the project directory:
- `<proj_dir>/README` contains instructions on working with layers, updating the git repo, etc.
- `<proj_dir>/layers/meta-yocto/meta-yocto-bsp/README.hardware contains details on the BSPs contained in the `meta-yocto` layer, which includes boot instructions for the beaglebone BSP.

### Step 3: Source environment variable scripts and Edit the `conf/local.conf` file
While still in the `beaglebone` directory source the two environment variable scripts in the order shown. This must be done every time you enter the project in a new bash shell, or switch to a different project in the same shell:
```
$ source ./environment-setup-x86_64-wrlinuxsdk-linux
$ source ./oe-init-build-env
```
The project can be customized in many ways by modifying the `conf/local.conf` file, but for now you must edit it to enable network access which is turned off by default. Use your favorite editor in place of `vim`:
```
$ vim conf/local.conf
```
Find the line which reads `BB_NO_NETWORK ?= '1'` and change the `1` to a `0`
```
   BB_NO_NETWORK ?= '1'
becomes:
   BB_NO_NETWORK ?= '0'
```
### Step 4: Build the Beaglebone Image
In the `beaglebone/build` directory from the last step, you can now start building the image with the `bitbake` command. You need to specify which image you want to build, see the WR Linux LTS 17 knowledge library for information on the content of different images.

Here is a quick summary of the most common images:
- `wrlinux-image-glibc-small` -- a minimal image with busybox
- `wrlinux-image-glibc-core` -- a minimal image without using busybox
- `wrlinux-image-glibc-std` -- a full featured headless image

To build a `glibc-std`build:
```
$ bitbake wrlinux-image-glibc-std
```
The build will take a while to build, depending on the speed of your machine.

### Where is the output?
The output of the build is located in
```
beaglebone/build/tmp-glibc/deploy/images/
```
### Appendix: Beaglebone BSP Notes from the meta-yocto README.hardware file
For convenience, here is the exact content of the current version of the README.hardware file located in 
`beaglebone/layers/meta-yocto/meta-yocto-bsp/README.hardware`:

```
Texas Instruments Beaglebone (beaglebone)
=========================================

The Beaglebone is an ARM Cortex-A8 development board with USB, Ethernet, 2D/3D
accelerated graphics, audio, serial, JTAG, and SD/MMC. The Black adds a faster
CPU, more RAM, eMMC flash and a micro HDMI port. The beaglebone MACHINE is
tested on the following platforms:

  o Beaglebone Black A6
  o Beaglebone A6 (the original "White" model)

The Beaglebone Black has eMMC, while the White does not. Pressing the USER/BOOT
button when powering on will temporarily change the boot order. But for the sake
of simplicity, these instructions assume you have erased the eMMC on the Black,
so its boot behavior matches that of the White and boots off of SD card. To do
this, issue the following commands from the u-boot prompt:

    # mmc dev 1
    # mmc erase 0 512

To further tailor these instructions for your board, please refer to the
documentation at http://www.beagleboard.org/bone and http://www.beagleboard.org/black

From a Linux system with access to the image files perform the following steps:

  1. Build an image. For example:

     $ bitbake core-image-minimal

  2. Use the "dd" utility to write the image to the SD card. For example:

     # dd core-image-minimal-beaglebone.wic of=/dev/sdb

  3. Insert the SD card into the Beaglebone and boot the board.
```
