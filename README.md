# Dockerize Cisco Nexus 9000v

The procedure to dockerize nxos changes pretty often (for example I wasn't able to use the procedure I was using on I7 with the new 9.3(3))  
so here is a repo that tries to stay up to date.

**Current state of this procedure** : Tested & Validated with **NX-OS 7.0.3.I7(9), NX-OS 9.3(3), NX-OS 9.3(7) and NX-OS 10.1(1)** and most specifically the **9300v** version.

Table of Contents
=================

   * [Dockerize Cisco Nexus 9000v](#dockerize-cisco-nexus-9000v)
      * [Download image from cisco.com](#download-image-from-ciscocom)
         * [Which image to choose ?](#which-image-to-choose-)
            * [Vagrant or KVM ?](#vagrant-or-kvm-)
            * [9300 or 9500 ?](#9300-or-9500-)
      * [Ok, got the image, what now ?](#ok-got-the-image-what-now-)
         * [Prepare the qcow2 image (from Vagrant box, skip this step if you got the KVM image)](#prepare-the-qcow2-image-from-vagrant-box-skip-this-step-if-you-got-the-kvm-image)
         * [Build the docker image (can't offer pre-built image because of Cisco's Licence)](#build-the-docker-image-cant-offer-pre-built-image-because-of-ciscos-licence)
      * [Run the docker image](#run-the-docker-image)


## Download image from cisco.com

With a (free) cisco.com account, you can download the image [here](https://software.cisco.com/download/home/286312239/type/282088129/release/9.3(3)).

### Which image should I choose ?

#### Vagrant or KVM ?

I decided to use **Vagrant** image because it comes with a backed config.

You can, of course, use the KVM image, but then you'll have to handle the POAP phase and the first config you want to apply.  
(**This procedure aim to be able to handle the POAP phase. Last time I tried it failed but I'll definitely try again very soon)**

#### 9300 or 9500 ?

I tried and validated only the 9300v image.  
I'm not very fond of the added complexity of a chassis so I won't test/validate the 9500v image.

## Ok, got the image, what now ?

### Prepare the qcow2 image (skip this step if you downloaded the "KVM" image and not the "Vagrant" one)

```bash
mkdir nxos
tar -xvf nexus9300v.9.3.3.box -C nxos/
cd nxos
qemu-img convert -f vmdk -O qcow2 box-disk1.vmdk nexus9300v.9.3.3.qcow2
```

### Build the docker image (can't offer pre-built image because of Cisco's Licence)

Here, we will use a modified fork of the amazing [vrnetlab](https://github.com/plajjan/vrnetlab) project.  
Had to tweak some options to make it work with the official 9.3(3) image.

I will maybe propose a PR to the main project when my modifications will be cleaner but for now :

```bash
git clone https://github.com/jpmondet/vrnetlab
mv nexus9300v.9.3.3.qcow2 vrnetlab/nxos/
cd vrnetlab/nxos/
make
```

/!\ Verify that you have, at least, 6 GB available on your `/var/lib/docker`. Yeah, this is huge... (The docker final image is actually "only" 2.35GB but  
you need more during the build to get the base image and pass the qcow2 to the docker context).


## Run and Use the docker image

`docker run -d --name nxos --privileged jpmondet/nxos:v933` (modify the name of the docker image to match yours)

(...Wait ~5min... You can use `watch -n 1 "docker ps"` to know when the container becomes **"Healthy"**)

Then, from another terminal :

`ssh admin@$(docker inspect --format '{{.NetworkSettings.IPAddress}}' nxos)`  (password is also _admin_)

or 

`telnet $(docker inspect --format '{{.NetworkSettings.IPAddress}}' nxos) 5000`

**Netconf port (830) & nxapi/restconf port (80) are also usable** (after having enable the features). You can even play with snmp.


Enjoy ! :-)
