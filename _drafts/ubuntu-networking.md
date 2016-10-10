Create an lxc container (Ubuntu 16.04) to test networking configuration:

    lxd-images import ubuntu --alias ubuntu-16.04 16.04
    lxc launch ubuntu-16.04 networking

[Ubuntu network configuration](https://help.ubuntu.com/lts/serverguide/network-configuration.html)

You can configure your network devices either with graphical utilites or command line.


> /etc/networks vs /etc/hosts


Static IP: vi /etc/network/interfaces

```
auto eth0
iface eth0 inet static
address 10.0.3.242
netmask 255.255.255.0
gateway 10.0.3.1
```

and then

    ifdown eth0
    ifup eth0


> Network Manager vs /etc/network ?

