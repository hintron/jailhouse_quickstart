Jailhouse Quickstart Guide
==
[*Jailhouse*][1] is an open source hypervisor based in Linux. It statically partitions the underlying hardware.

This quickstart guide will help you get [Jailhouse 0.8][2] up and running in QEMU quicker than any other guide. This expounds on the [Jailhouse readme][1].

# Requirements

* A [Digital Ocean][3] account and the ability to install a droplet with at least 2 vCPUs with an [Ubuntu][4] 17.10 x64 distro

* For Jailhouse to work, QEMU needs to be >= v2.7 and Linux need to be >= v4.4. Both of these are satisfied with Ubuntu 17.10. The guest image also needs to be >v3.9.

# Create a Droplet

Log in to [Digital Ocean][3].

Create a droplet with an Ubuntu 17.10 x64 distro. Choose any droplet size with at least 2 vCPUs (for the cheapest plan, look under flexible plans for a 2 GB RAM, 2 vCPU plan for $15/month).

Choose any datacenter region and any additional options.

Name the hostname `ubuntu-17`.

Create the droplet.

SSH into the droplet as `root` using the credentials emailed to you. It will prompt you to change your password.

The current directory should be /root/

# Configure the Droplet

Get package stuff up to date:

    apt-get upgrade
    apt-get update

Install all the goodies we need:

    apt-get install -y git vim build-essential qemu

QEMU should be at version 2.10.

Download the guest Ubuntu image (server version):

    wget http://releases.ubuntu.com/17.10/ubuntu-17.10.1-server-amd64.iso

Create a compressed `qcow2` version of the iso image for QEMU:

    qemu-img convert -c ubuntu-17.10.1-server-amd64.iso ubuntu-17.10.1-server-amd64.qcow2 -O qcow2

You can remove the original iso file if you would like.

Make sure the `kvm-intel` module is loaded via the `lsmod` command. You should see a module called `kvm` that is used by `kvm_intel` and then the `kvm_intel` module itself.

If it's not loaded, load it like so (UNTESTED):

    insmod kvm-intel nested=1

Enable X forwarding, so the QEMU window can be forwarded through your SSH terminal:



# Starting QEMU


    qemu-system-x86_64 \
        -m 500M \
        -smp 2 \
        -enable-kvm \
        -machine q35,kernel_irqchip=split \
        -device intel-iommu,intremap=on,x-buggy-eim=on \
        -cpu kvm64,-kvm_pv_eoi,-kvm_steal_time,-kvm_asyncpf,-kvmclock,+vmx \
        -drive file=/root/ubuntu-17.10.1-server-amd64.qcow2,format=qcow2,id=disk,if=none \
        -device ide-hd,drive=disk \
        -serial stdio \
        -serial vc \
        -netdev user,id=net \
        -device intel-hda,addr=1b.0 \
        -display vnc=1 \
        -device hda-duplex
        #-display curses \
        #-device e1000e,addr=2.0,netdev=net \

        # To get out of curses, do ESC-1 or ESC-2 and type `quit`

TODO: I'm running into an issue where I can't get the QEMU window to forward properly. I keep getting this error: `Could not initialize SDL(No available video device) - exiting`.

[1]: https://github.com/siemens/jailhouse "Jailhouse"
[2]: https://github.com/siemens/jailhouse/releases "Jailhouse Releases"
[3]: https://www.digitalocean.com/ "Digital Ocean"
[4]: https://www.ubuntu.com/ "Ubuntu"
