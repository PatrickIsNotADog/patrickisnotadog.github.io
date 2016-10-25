# LVM

## LVM provides *dynamic partitions*, meaning that you can create, resize or delete LVM partitions, while your linux system is running. Other features it provides are:

* you can extend a logical volume over more than one disks.
* you can set up striped logical volumes, so that I/O can be distributed to all disks hosting the logical volume in parallel.
* you can take and rollback to snapshots.

Of course all these come to the price of:

* extra complexity.
* learning curve.

## Anatomy of LVM

Volume Group: the highest level abstraction within LVM, that gathers together a collection of Logical Volumes and Physical Volumes into one administrative unit.

Physical Volume: a hard disk or something that looks like a hard disk, like a software raid device.

Logical Volume: the equivalent of a disk partition in a non-LVM system. The Logical Volume is visible as a standard block device and as such it can contain a file system (e.g. /home).

Physical Extent: each physical volume is divided into chunks of data, known as physical extents, that have the same size as the logical extents for the volume group.

Logical Extent: each logical volume is split into chunks of data, known as logical extents. The extent size is the same for all logical volumes in the volumes group.

A virtual way to see all these is the following:

    hda1   hdc1      (PV:s on partitions or whole disks)                        
       \   /                                                                    
        \ /                                                                     
       diskvg        (VG)                                                       
       /  |  \                                                                  
      /   |   \                                                                 
  usrlv rootlv varlv (LV:s)
    |      |     |                                                              
 ext2  reiserfs  xfs (filesystems)                                        

## Steps when using LVM

1. Install required packages:

        apt-get install lvm2 thin-provisioning-tools
1. Initialize a disk or a partition for use by LVM:

        pvcreate ${disk path} (check if pvs returns volume)
1. Create a volume group. You can use the default 32MB extent size, or declare a custom one using the `-s` flag. The command to create the volume group is:
        vgcreate ${volume group} ${disk(s)} (check if vgs returns a volume group)
1. Create a logical volume LXDPool that will be used by lxd as a logical volume pool; the advantage of manually creating in this step is that you can configure its size, while lxd will greedy get all available volume group size and assign it to the pool:

        lvcreate -L 20G --thinpool LXDPool lxd
1. You can configure LXD to use an LVM volume group called **lxd** for containers and images, with the command:

        lxc config set storage.lvm_vg_name lxd
    Now LXD will create a logical volume for each image it contains. When launching a new container, its rootfs will start as a clone of the corresponding image's logical volume. Container snapshots are also created as logical volume snapshots. You can also set the thinpool's name and the logical volume size:

        lxc config set storage.lvm_thinpool_name LXDPool
        lxc config set storage.lvm_volume_size 40




## Example

Create an extra disk and attach it to the VM:

    qemu-img create -f qcow2 lvm-demo-vdc.qcow2 100G
     virsh attach-disk lvm-demo --source `pwd`/lvm-demo-vdc.qcow2 --target vdc --driver qemu --subdriver qcow2 --persistent


## Nested LVM

1. pvcreate /dev/vdc
1. vgcreate vg00 /dev/vdc
1. lvcreate -L 80G --name lxd vg00
1. pvcreate /dev/vg00/lxd
1. vgcreate lxd /dev/vg00/lxd

## Resize an existing Logical Volume

    lvextend -L${new size} --resizefs ${logical volume}

The option `--resizefs` is needed if you have a container using the LV as its root filesystem, so that the underlying fs gets updated too.

         
