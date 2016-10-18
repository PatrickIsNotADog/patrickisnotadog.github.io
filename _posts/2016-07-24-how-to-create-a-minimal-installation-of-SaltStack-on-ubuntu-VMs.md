In this post I will describe how you can create a SaltStack infrastructure with one master and two minion nodes, using VMs created with [libvirt](https://libvirt.org), [QEMU](http://wiki.qemu.org) and [KVM](http://www.linux-kvm.org).

## Virtualization Tools

Before I present the commands that you have to run to create the VMs, I want to spend a little time talking about the tools I am going to use.

### KVM

Kernel-based Virtual Machine is a virtualization infrastructure, merged into the Linux kernel, that can
turn it into an hypervisor. KVM requires a processor with hardware-assisted virtualization extension. It does not perform any emulation itself, but exposes the */dev/kvm* interface instead, that a userspace host can leverage to  run multiple virtual machines, running unmodified Linux or Windows images. 

### QEMU

QEMU is generic and open source machine emulator and virtualizer. It can work without ay need for hardware virtualization support, but when integrated with Xen or KVM hypervisors it can achieve near native performance for CPUs.

### Libvirt

Libvirt is a collection of software that provides a convenient way to manage virtual machines and other virtualization functionality, such as storage and network interface management. Its primary goal is to provide a single way to manage multiple different hypervisors, so you don't have to learn the hypervisor specific tools. For example, the command `virsh list --all` can be used to list the existing virtual machines for any supported hypervisor (KVM, Xen, VMWare ESX, etc).

## Create the VMs

Now, I am ready to describe the steps required to create the VMs. I suppose you already have a Linux distribution on your PC and I will continue my description with the commands I will run on my Ubuntu 16.04 LTS host.

First of all, you have to install the required packages, described above. But since a KVM solution is going to be used, you have to be sure that your processor supports the necssary virtualization extensions. First, install the package *cpu-checker* by running:

    sudo apt-get install cpu-checker
    
and then just type:

    kvm-ok
    
A message will be printed, informing you whether your CPU supports hardware virtualization or not. *KVM acceleration can be used*? Nice! Continue with:

    sudo apt install qemu-kvm libvirt-bin virtinst

to install libvirt, as described in [Ubuntu's documentation for libvirt](https://help.ubuntu.com/lts/serverguide/libvirt.html). You can also install the GUI with `sudo apt install virt-manager`, but I feel like doing it all from the console right now, so I will skip this step.

Now that all the packages are installed, you can download an Ubuntu cloud image, from [the corresponding repo](https://cloud-images.ubuntu.com/). Cloud images are images designed to be used in cloud environments and this is why they are relatively small and perform some initial system configuration tasks when booted, using metadata provided by the cloud environment. These two factors make them attractive for use outside of cloud environments too, like this virtualization example.

Inside the *code* sub-directory of this project, you can find the *create_vm.sh* script, that automates the procedure of converting the Ubuntu cloud image to qcow2 format, creating the ISO image with the required *user-data* and *meta-data* files that will be used in the initial system configuration tasks described above and creates the VM using libvirt. The first lines of the script should be edited to contain the location of the cloud image downloaded and the name of the VM to be created:

```bash
# The cloud image that will be used to create the VM
cloud_image=/home/${USER}/Downloads/xenial-server-cloudimg-amd64-disk1.img

# VM's name
vmname="demo"
```

Also, ssh keys will be used for access to the virtual machines, so if you have no keys in your `~/.ssh` directory, you can create them with the command:

    ssh-keygen -t rsa

Run the script three times, to create the VMs with names:

* master
* minion-01
* minion-02

and run 

    virsh list --all

to make sure the VMs were created. You can start the  VMs with

```bash
virsh start master
virsh start minion-01
virsh start minion-02
```

I do not provide static IPs to the VMs, but you can run

    arp -na 

to see the IPs that have been registered for libvirt's default *virbr0* interface and then connect with

    ssh ubuntu@${VM IP}



## Configure minions

Now that the VMs are running and you have access to them, connect to *minion-01* and follow the instructions to install the required packages as described in [SaltStack documentation](https://repo.saltstack.com/#ubuntu). The project has great documentation!

The steps required for Ubuntu 16.04 are:

1. Run the following command to import the SaltStack repository key: 

        wget -O - https://repo.saltstack.com/apt/ubuntu/16.04/amd64/latest/SALTSTACK-GPG-KEY.pub | sudo apt-key add -
1. Save the following file to `/etc/apt/sources.list.d/saltstack.list`:

        deb http://repo.saltstack.com/apt/ubuntu/16.04/amd64/latest xenial main
1. Run `sudo apt-get update`.
1. Install salt-minion with `sudo apt-get install salt-minion`.
1. Start salt-minion service with: `sudo systemctl start salt-minion	`.
1. Edit */etc/salt/minion* file to include the line `master: ${master IP}`, where *${master IP}* is the IP attached to *master* VM.

Now you have a SaltStack minion node, that will seek for its master. Repeat for *minion-02*.


## Configure master

Following the SaltStack documentation you have to install `salt-master` package in *master* VM to configure it as a SaltStack master node. Most steps are similar to the minion nodes, described above.

Once the node is configured as a SaltStack master, run `salt-key -L`. The output should be the following:

```
root@master:/home/ubuntu# salt-key -L
Accepted Keys:
Denied Keys:
Unaccepted Keys:
minion-01
minion-02
Rejected Keys:
```

This means that the minion nodes have found their master and send their keys for acceptance. To accept all keys run `salt-key -A`. 

Congratulations! You have now a SaltStack infrastructure with a master and two minions connected to him. You can run `salt "*" test.ping` to check connectivity. The result should be the following:

```
minion-01:
    True
minion-02:
    True
```
