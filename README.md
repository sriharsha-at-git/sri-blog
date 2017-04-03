# Node.js Starter Overview

The Node.js Starter demonstrates a simple, reusable Node.js web application based on the Express framework.

## Run the app locally

1. [Install Node.js][]
2. Download and extract the starter code from the Bluemix UI
3. cd into the app directory
4. Run `npm install` to install the app's dependencies
5. Run `npm start` to start the app
6. Access the running app in a browser at http://localhost:6001

[Install Node.js]: https://nodejs.org/en/download/

#######################################################################
 MOVE to LXD
 ######################################################################
 
 Ubuntu 16.04 LTX (Xenial Xersus)
########################################################################################################################
					Installing and configuring LXD
########################################################################################################################
### Update system
sudo apt update
sudo apt upgrade

### Install lxd & zfs
sudo apt-get install lxd
sudo apt-get install zfs-utils
# OR sudo apt install zfs lxd
sudo reboot

### configure lxd/Initialize lxc daemon
sudo lxd init

### Network configuration
lxc config set core.https_address [::]
lxc config set core.trust_password some-secret-string
# or add client certificate
lxc config trust add client.crt

### proxy configuration
lxc config set core.proxy_http http://proxy-url:port
lxc config set core.proxy_https http:///proxy-url:port
lxc config set core.proxy_ignore_hosts image-server.local

### image management
# cache image for 5 days since last used(default - 10 days)
lxc config set images.remote_cache_expiry 5
# look fo rimage updates every 24 hours set when –auto-update flag is used (default - 6 hours)
lxc config set images.auto_update_interval 24
# no auto updte for cached images
lxc config set images.auto_update_cached false

########################################################################################################################
					LXD container & management
########################################################################################################################

### launching machine/system containers
# launches a ubuntu stable srelease 16.04 LTS
lxc launch ubuntu:
# creates a ubuntu images with random naame assigned
lxc launch ubuntu:14.04
# with a custom name (alias) - c1
lxc launch ubuntu:14.04 c1
# for 32 bit releases-(i386)
lxc launch ubuntu:14.04/i386 c2
# for ubuntu developmental releases
lxc launch ubuntu-daily:devel c3
lxc launch ubuntu-daily:xenial c4
# for latest alpine  64 bit (amd64)
lxc launch images:alpine/3.3/amd64 c5

### List images default/local/remote

lxc image list ubuntu:
lxc image list ubuntu-daily:
# unofficial images- remote repo
lxc image list images:
# filter by os name/preffix
lxc image alias list ubuntu:


### Create containers
# without launching/running
lxc init ubuntu:


### Information about container's
#Listing the containers
lxc list
# fast display
lxc list --fast
# more filetrs:--
lxc list security.privileged=true
lxc list --fast alpine
lxc list --fast ubuntu

# detailed information
lxc info <container_name>


### Life cycle management
lxc start <container_name>
lxc stop <container_name>
lxc stop <container_name> --force
lxc restart <container_name>
lxc pause <container_name>
lxc delete <container_name>

### Container configuration can be performed at these ...
disk
nic
unix-block (physical blocks like dev/sda/sdb)
unix-char (virtual blocks like kvm/lvm)
none

### Configuration profiles
# listprofiles
lxc profile list
# show details of specific profile
lxc profile show <profile>
# edit profile details
lxc profile edit <profile>
# apply multiple profiles to container/s
lxc profile apply <container> <profile1>,<profile2>,<profile3>,...

### Local configuration (specific to a container/s)
# edit a container directly; just like “profile edit”
lxc config edit <container>
# adding specific key-value pairs
lxc config set <container> <key> <value>
# adding deivces ; Which will setup a /dev/kvm entry for the container named “my-container”.
# The same can be done for a profile using “lxc profile set” and “lxc profile device add”.
lxc config device add my-container kvm unix-char path=/dev/kvm

### Reading the configuration
# read local container configuration
lxc config show <container>
# expanded view
lxc config show --expanded <container>

### Live configuration update
# All configuration chnages are applied while the containe/s are running; I.E no restart required

### Execution environment
# get to a shell inside a container
lxc exec <container> bash
# multiple commands passed (/) without ssh'n 
lxc exec <container> -- ls -lh /
# setting/ overriding env variables
lxc exec <conatiner> --env mykey=myvalue env | grep mykey

### Managing files
# pulla file form conatines
lxc file pull <conatiner>/etc/hosts hosts
# push a file to conatiner
lxc file push <container>setup.sh /home/user/
# edit a file directly inside a container
lxc file edit <container>/<path>

### Snapshot management
# create a snapshot of container with alias as snapX X is a incremental number
lxc snapshot <container>
# with a psudo/alias name
lxc snapshot <container> <snapshot name>
# list snapshots for a container
lxc info <container>
# restore a snapshot
lxc restore <container> <snapshot name>
# rename a snapshot
lxc move <container>/<snapshot_name> <container>/<new_snapshot_name>
# create new container from snapshot
lxc copy <source_container>/<snapshot_name> <destination_container>
# delete container
lxc delete <container>/<snapshot_name>

## Cloning and renaming containers
# copy a container and effectively; with MAC address reset
lxc copy <source_container> <destination_container>
# rename container OR moving a container b/w hosts
lxc move <old_name> <new_name>

########################################################################################################################
					LXC Container Resource control's
########################################################################################################################

# Available resource limits are specified at container level or at profile level.
Disk/CPU/Memory/Network-I/Block-IO

lxc config set CONTAINER KEY VALUE
lxc profile set PROFILE KEY VALUE
lxc config device set CONTAINER DEVICE KEY VALUE
lxc profile device set CONTAINER DEVICE KEY VALUE

## cpu's
# limiting number of processors
lxc config set my-container limits.cpu 2
# pinning cpu core's
lxc config set my-container limits.cpu 1,3
# more complex - setting ranges in between
lxc config set my-container limits.cpu 0-3,7-11

# limit the CPU time of a container to 10% of the total
lxc config set my-container limits.cpu.allowance 10%
# give a fixed slice of cpu
lxc config set my-container limits.cpu.allowance 25ms/200ms
# to reduce the priority of a container to a minimum
lxc config set my-container limits.cpu.priority 0

## memory
# straightforward memory limit
lxc config set my-container limits.memory 256MB
# turn off swap memory (default is enabled)
lxc config set my-container limits.memory.swap false
# tell the kernel to swap this container’s memory first:
lxc config set my-container limits.memory.swap.priority 0
#  no hard memory limit enforcement
lxc config set my-container limits.memory.enforce soft

## Disk and block I/O
# set a disk limit (requires btrfs or ZFS):
lxc config device set my-container root size 20GB
# to restrict speed
lxc config device set my-container limits.read 30MB
lxc config device set my-container limits.write 10MB
# to restrict Iops
lxc config device set my-container limits.read 20Iops
lxc config device set my-container limits.write 10Iops
# To increase the I/O priority for that container to the maximum; for a busy system with over-commit
lxc config set my-container limits.disk.priority 10

## Network I/O
# Network I/O is identical to block I/O as far the knobs available
lxc exec zerotier -- wget http://speedtest.newark.linode.com/100MB-newark.bin -O /dev/null
lxc profile device set default eth0 limits.ingress 100Mbit
lxc profile device set default eth0 limits.egress 100Mbit
# set an overall network priority
lxc config set my-container limits.network.priority 5

## Getting the current resource usage
# gets MEMORY, DISK & NETWORK
lxc info <container_name>


########################################################################################################################
					Image management 
########################################################################################################################

### Container images - every LXD container is created from a local image.
# containers can be created/imported using its full hash, short hash or an alias

# the remote image import to local LXD image store as a cached image, then the container will be created from it
lxc launch ubuntu:14.04 c1
# next time; LXD will only check that the image is still up to date (when not referring to it by its fingerprint), if it is, it will create the container without downloading anything.
# call by full hash
lxc launch ubuntu:75182b1241be475a64e68a518ce853e800e9b50397d2f152816c24f038c94d6e c2
# call by short hash
lxc launch ubuntu:75182b1241be c3
lxc launch 75182b1241be c4
# create from own local image under the name “myimage"
lxc launch my-image c5

## Manually importing images

# Copying from an image server
lxc image copy ubuntu:14.04 local:
# copy with a alias name
lxc image copy ubuntu:12.04 local: --alias old-ubuntu
# launch with alias name
lxc launch old-ubuntu c6

# just use the aliases that were set on the source server; copy the aliases
lxc image copy ubuntu:15.10 local: --copy-aliases
lxc launch 15.10 c7

# keep the image up to date, request it with the –auto-sync flag;
lxc image copy images:gentoo/current/amd64 local: --alias gentoo --auto-update

# Importing a tarball
lxc image import <tarball>
# set an alias at import time
lxc image import <tarball> --alias random-image

# For two tarballs; import multiple tarballs
lxc image import <metadata tarball> <rootfs tarball>

# Importing from a URL; 
lxc image import https://dl.stgraber.org/lxd --alias busybox-amd64

## Managing the local image store

# Listing images
lxc image list
# filter based on the alias or fingerprint
lxc image list amd64
# filter by image properties: specifying a key=value
lxc image list os=ubuntu
# information about a given image
lxc image info ubuntu

# Editing images properties and some of the flags 
lxc image edit <alias or fingerprint>

# Deleting images
lxc image delete <alias or fingerprint>

# Exporting images to local image store
lxc image export old-ubuntu .

# Image formats
Unified image (single tarball)
Split image (two tarballs)

## Creating own images
# Turning a container into an image
lxc launch ubuntu:14.04 my-container
lxc exec my-container bash
<do whatever change>
lxc publish my-container --alias my-new-image
# turn a past container snapshot into a new image
lxc publish my-container/some-snapshot --alias some-image

## Manually building an image
1. Generate a container filesystem. This entirely depends on the distribution you’re using. For Ubuntu and Debian, it would be by using debootstrap.
2. Configure anything that’s needed for the distribution to work properly in a container (if anything is needed).
3. Make a tarball of that container filesystem, optionally compress it.
4. Write a new metadata.yaml file based on the one described above.
5. Create another tarball containing that metadata.yaml file.
6. Import those two tarballs as a LXD image with: -
lxc image import <metadata tarball> <rootfs tarball> --alias some-name

## Publishing your images
All LXD daemons act as image servers. 
Unless told otherwise all images loaded in the image store are marked as private and 
so only trusted clients can retrieve those images, but should you want to make a public image server, 
all you have to do is tag a few images as public and make sure you LXD daemon is listening to the network.

# running a public LXD server
# share LXD images is to run a publicly visible LXD daemon
lxc config set core.https_address "[::]:8443"
# For remote users :: add server as a public image server with:
lxc remote add <some name> <IP or DNS> --public

## Use a static web server ???
## Build a simplestreams server ???


########################################################################################################################
					Remote hosts and container migration
########################################################################################################################

# Supported Remote protocols
LXD 1.0 API
Simplestreams (read-only, image-only protocol)

# Adding a remote
lxc config set core.https_address [::]:8443
lxc config set core.trust_password something-secure

lxc remote add repo ip.add.ip.add(1.2.3.4)
# list your remotes
lxc remote list

# Interacting with remote images
lxc launch ubuntu:14.04 c1
lxc launch ubuntu:14.04 foo:c1

# Listing running containers on a remote host 
lxc list foo:
# create a container from remote image store use repo:image_name
lxc launch foo:my-image foo:c2
# getting a shell into a remote container
lxc exec foo:c1 bash

# Copying containers
lxc copy foo:c1 c2
Requres a stop of source container; instead just copy a snapshot and do it while the source container is running:
lxc snapshot foo:c1 current
lxc copy foo:c1/current c3

# Moving containers
# stop the source container prior to moving it
lxc stop foo:c1
lxc move foo:c1 local:
## OR
lxc stop foo:c1
lxc move foo:c1 c1

########################################################################################################################
					Docker in LXD
########################################################################################################################

## Running a basic Docker workload
# launching a Ubuntu 16.04 container
lxc launch ubuntu-daily:16.04 docker -p default -p docker
# make container up to date and install docker
lxc exec docker -- apt update
lxc exec docker -- apt dist-upgrade -y
lxc exec docker -- apt install docker.io -y
# thats it, run the desired docker container app
lxc exec docker -- docker run --detach --name app carinamarina/hello-world-app
lxc exec docker -- docker run --detach --name web --link app:helloapp -p 80:5000 carinamarina/hello-world-web

# With those two Docker containers now running, we can then get the IP address of our LXD container and access the service!
lxc list
# above results with ip=10.178.150.73, now test it
curl http://10.178.150.73

# If workload doesn’t work properly; then we could trust the user inside the LXD container
lxc config set docker security.privileged true
lxc restart docker


########################################################################################################################
					LXD in LXD
########################################################################################################################

### Nesting LXD
# start an Ubuntu 16.04 container with nesting enabled:
lxc launch ubuntu-daily:16.04 c1 -c security.nesting=true
# set the security.nesting key on an existing container
lxc config set <container name> security.nesting true

# for all containers using a particular profile
lxc profile set <profile name> security.nesting true

# With that container started, you can now get a shell inside it, configure LXD and spawn a container:
lxc launch ubuntu-daily:16.04 c1 -c security.nesting=true
lxc exec c1 bash
root@c1:~# lxd init
### result -  Name of the storage backend to use (dir or zfs): dir.......
root@c1:~# lxc launch ubuntu:14.04 trusty
root@c1:~# lxc list

### The online demo server
# setup own similar server, you can grab the code for our website and the daemon with:
git clone github.com/lxc/linuxcontainers.org
git clone github.com/lxc/lxd-demo-server

########################################################################################################################
					Live migration
########################################################################################################################

########################################################################################################################
					LXD and Juju
########################################################################################################################

########################################################################################################################
					LXD and OpenStack 
########################################################################################################################

### "conjure-up” to deploy a full Ubuntu OpenStack using Juju

# Requirements
Working LXD setup, providing containers with network access and 
that you have a pretty beefy CPU, around 50GB of space for the container to use and at least 16GB of RAM.
....
....
....
....

########################################################################################################################
					Debugging and contributing to LXD
########################################################################################################################

