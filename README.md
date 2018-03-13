Jailhouse Quickstart Guide
==
[*Jailhouse*][1] is an open source hypervisor based in Linux. It statically partitions the underlying hardware.

This quickstart guide will help you get [Jailhouse 0.8][2] up and running in QEMU quicker than any other guide. This expounds on the [Jailhouse readme][1].

# Requirements

* A [Digital Ocean][3] account and the ability to install a droplet with at least 2 vCPUs with an [Ubuntu][4] 17.10 x64 distro.

* Intel CPUs. Digital Ocean should be using Intel CPUs, but to double check, run `less /proc/cpuinfo`.

* For Jailhouse to work, the host Linux image needs to have a kernel >= v4.4 and for QEMU to be >= v2.7. Both of these are satisfied with Ubuntu 17.10. The guest image running *inside* QEMU needs to have a Linux kernel > v3.9. Debian 9.4.0 Stretch satisfies this requirement.

* X11 forwarding enabled over SSH. I use [MobaXterm][5] in Windows as my SSH client.

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

Set the correct timezone:

    dpkg-reconfigure tzdata

Download the guest image to run in QEMU (Debian 9.4.0 - Stretch, with kernel v4.9):

    wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-9.4.0-amd64-netinst.iso

Create a 10 GB vm qcow2 img to install the guest to:

    qemu-img create -f qcow2 debian_9_4_0.qcow2 10G

If you happen to have an existing img file, convert to qcow2 like so:

    qemu-img convert -c -f raw -O qcow2 debian_9_4_0.img debian_9_4_0.qcow2

Make sure the `kvm-intel` module is loaded via the `lsmod` command. You should see a module called `kvm` that is used by `kvm_intel` and then the `kvm_intel` module itself.

Also, make sure that `kvm-intel` is loaded with the `nested=1` parameter. To do this, do

    less /sys/module/kvm_intel/parameters/nested

There should be a `Y` in there, indicating that the nested param is 1.

If it's not loaded, load it like so (UNTESTED):

    insmod kvm-intel nested=1

# Installing a Debian guest OS in QEMU

Load the installer image as a cd-rom in qemu:

    qemu-system-x86_64 \
        -hda /root/debian_9_4_0.qcow2 \
        -boot d \
        -cdrom /root/debian-9.4.0-amd64-netinst.iso

If you run into an error where QEMU quits with `Could not initialize SDL(No available video device) - exiting` then see [Getting QEMU to forward graphical windows over SSH](#getting-qemu-to-forward-graphical-windows-over-ssh).

Once the installer loads, choose the `Install` option.

Run through all the defaults. When it comes to software to install, only select standard system utilities.

When the installer finishes and tries to reboot, exit the QEMU window. Run QEMU again, this time removing the installation image 'disk':

        qemu-system-x86_64 -hda /root/debian_9_4_0.qcow2

Login as `root` into the QEMU guest (which will be the jailhouse host) and install necessary packages:

    apt-get install -y git build-essential linux-headers-$(uname -r)

`linux-headers-$(uname -r)` will return the exact linux headers necessary (e.g. `linux-headers-4.9.0-6-amd64`). You can manually search for it like so: `apt-cache search linux-headers-$(uname -r)`.

Linux headers are necessary, since Jailhouse is a Loadable Kernel Modul (LKM), and to build LKMs, you need the correct Linux headers.

# Install Jailhouse

Download, extract, and compile jailhouse 0.8:

    wget https://github.com/siemens/jailhouse/archive/v0.8.tar.gz
    tar xvf v0.8.tar.gz
    cd jailhouse-0.8
    make

Once built, install jailhouse:

    make install

# Conclusion

TODO: Now what? How do I run the QEMU examples?



# Full QEMU command:

TODO: When I install linux, will things get messed up if I don't specify all these extra VM params from the beginning? Does linux need to be installed knowing all these CPU features, or can they just be turned on when needed? I guessing it's the latter... so I may have to reinstall linux later.

    qemu-system-x86_64 \
        -m 1G \
        -smp 2 \
        -enable-kvm \
        -machine q35,kernel_irqchip=split \
        -cpu kvm64,-kvm_pv_eoi,-kvm_steal_time,-kvm_asyncpf,-kvmclock,+vmx \
        -device intel-iommu,intremap=on,x-buggy-eim=on \
        -serial stdio \
        -serial vc \
        -drive file=debian_9_4_0.qcow2,format=qcow2,id=disk,if=none \
        -device ide-hd,drive=disk \
        -device intel-hda,addr=1b.0 \
        -device hda-duplex

        #-netdev user,id=net \
        #-device e1000e,addr=2.0,netdev=net \


# Getting QEMU to forward graphical windows over SSH

If you run into an error where QEMU quits with `Could not initialize SDL(No available video device) - exiting` then do the following workaround to "lubricate" the X11 forwarding mechanism:

Exit the current terminal session and re-ssh into the terminal and try again. If that doesn't work, install and run firefox:

    apt-get install firefox
    firefox

There should be a Firefox GUI that is forwarded to you. If so, QEMU should now be able to work. Whatever Firefox does gets things working.

Back to [Installing a Debian guest OS in QEMU](#installing-a-debian-guest-os-in-qemu).

[1]: https://github.com/siemens/jailhouse "Jailhouse"
[2]: https://github.com/siemens/jailhouse/releases "Jailhouse Releases"
[3]: https://www.digitalocean.com/ "Digital Ocean"
[4]: https://www.ubuntu.com/ "Ubuntu"
[5]: https://mobaxterm.mobatek.net/ "MobaXterm"
