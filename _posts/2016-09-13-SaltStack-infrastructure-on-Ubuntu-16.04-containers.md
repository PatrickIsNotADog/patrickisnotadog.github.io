At [a previous post](../saltstack-minimal-installation/README) I described how you can create a minimal SaltStack installation with a master and two minion nodes, running on KVM VMs. Now, I will describe the same procedure using linux containers; the steps to achieve a proper working installation are few and simple.

First, install lxd package, if it is not already installed:

    sudo apt-get install lxd

[LXD](https://linuxcontainers.org/lxd) is a project founded and currently led by [Canonical](http://www.canonical.com), that is built on top of LXC, to provide a more friendly user experience, network control on the containers through RESTful services and an OpenStack plugin. [LXC](https://linuxcontainers.org/lxc) is a project under the [Linux Containers](https://linuxcontainers.org) umbrella (same as LXD), and provides the communication with the Linux kernel, that enables users to create and manage linux containers. Since Ubuntu 16.04, LXD is the default package for linux containers; although I have not used it over the network, I also prefer LXD due to its more user friendly CLI client API.

Once LXD is installed on your machine, you can manually download Linux images from *remotes*, which are servers dedicated to host and provide Linux images for LXD. In my machine I found the following preinstalled *remotes*:

```bash
alex@travelling:~$ lxc remote list
+-----------------+------------------------------------------+---------------+--------+--------+
|      NAME       |                   URL                    |   PROTOCOL    | PUBLIC | STATIC |
+-----------------+------------------------------------------+---------------+--------+--------+
| images          | https://images.linuxcontainers.org       | lxd           | YES    | NO     |
+-----------------+------------------------------------------+---------------+--------+--------+
| local (default) | unix://                                  | lxd           | NO     | YES    |
+-----------------+------------------------------------------+---------------+--------+--------+
| ubuntu          | https://cloud-images.ubuntu.com/releases | simplestreams | YES    | YES    |
+-----------------+------------------------------------------+---------------+--------+--------+
| ubuntu-daily    | https://cloud-images.ubuntu.com/daily    | simplestreams | YES    | YES    |
+-----------------+------------------------------------------+---------------+--------+--------+

```

In order to download the Ubuntu 16.04 image, you can

* use *images* remote, which is linked to the [public LXD image server](https://images.linuxcontainers.org), that contains unofficial images, with the syntax `lxc image copy images:/ubuntu/xenial/amd64 local: --alias ubuntu-16.04`, or
* use the official *ubuntu remote*, with the syntax `lxc image copy ubuntu:16.04 local: --alias ubuntu-16.04`.

If *images remote* is not already configured you can run:

    lxc remote add images images.linuxcontainers.org


Verify with the image is now available to your host:

```bash
alex@travelling:~$ lxc image list
+--------------+--------------+--------+------------------------------------+--------+----------+--------------------------------+
|    ALIAS     | FINGERPRINT  | PUBLIC |            DESCRIPTION             |  ARCH  |   SIZE   |          UPLOAD DATE           |
+--------------+--------------+--------+------------------------------------+--------+----------+--------------------------------+
| ubuntu-16.04 | f452cda3bccb | no     | Ubuntu 16.04 LTS server (20160627) | x86_64 | 138.23MB | Jul 14, 2016 at 10:44pm (EEST) |
+--------------+--------------+--------+------------------------------------+--------+----------+--------------------------------+
```

Launch the master and minion instances

```bash
lxc launch ubuntu-16.04 master
lxc launch ubuntu-16.04 minion-01
lxc launch ubuntu-16.04 minion-02
```

and verify with

```bash
alex@travelling:~$ lxc list
+-----------+---------+------------+------+-----------+-----------+
|   NAME    |  STATE  |    IPV4    | IPV6 | EPHEMERAL | SNAPSHOTS |
+-----------+---------+------------+------+-----------+-----------+
| master    | RUNNING | 10.0.3.165 |      | NO        | 0         |
| minion-01 | RUNNING | 10.0.3.138 |      | NO        | 0         |
| minion-02 | RUNNING | 10.0.3.214 |      | NO        | 0         |
+-----------+---------+------------+------+-----------+-----------+
```

> Alternatively, you can first launch a container with `lxc launch ubuntu:16.04 master` for example, that will retrieve the Ubuntu 16.04 image and then launch the container. If you run `lxc image list`, you will see that the image is saved, without an alias and a fingerprint is assigned to it. In order to add an alias, you can `lxc image alias create ubuntu-16.04 ${FINGERPRINT}`, and confirm with `lxc image list`.

Here you can see the names, state and also IP address assigned to the instances by LXD.

Now you can use

    lxc exec ${instance name} /bin/bash

to connect to each of the instances and follow the [SaltStack minimal installation instructions](../saltstack-minimal-installation) to properly configure the instances as master and minion nodes.

In a few minutes you should be able to see the following result

```
root@master:~# salt "*" test.ping
minion-01:
    True
minion-02:
    True
```

Simple, wasn't it? Linux containers rock! \m/


