# Tourlou

Tourlou, written *τουρλού* in greek, or *türlü* in the neighbour Turkey, is a food made of various different summer vegetables. When used twice, as *τουρλού τουρλού*, it obtains the metaphorical meaning of an irregural, usually unmatched, crowded mix of people or artifacts.

![tourlou](../img/tourlou.jpg)

I started working on an OpenStack project some months ago and not having been part of a DevOps team before, I used to hear a looot of new words, or terms I had come across with, but never searched their exact meaning. In order fill the gap between my dev knowledge and the admin territory, I came up with a practical solution: I created a list in my smartphone and every time I was hearing a new term, I wrote it down. When I had free time, I would google my words; sometimes more and sometimes less thoroughly. But I did not write anything down, something I later regretted. In this post I will save all the terms I search, no matter how irrelevant they may be with each other, no matter how imperfect or subjective they may be.

## Reverse Proxy

*Reverse Proxy* is a proxy that receives requests on behalf of one or more backend servers, that will actually serve the requests. In this way it hides the complexity of the actual server infrastracture and implementation, while it can be used to add an extra security layer, or to provide high availability and load balancing.

## Forward Proxy

On the other hand, a *Forward Proxy* is a client side concept, that hides one or more clients behind a proxy entity, that acts for their behalf. A common *Forward Proxy* is a home router, that routes web traffic for all devices connected to it, masqueraded under NAT. This way, user anonimity can be achieved.

> TODO: create diagrams

## Iptables

Iptables is a firewall, used on Linux distros to hanlde and instruct rules on IP traffic. In the official manual page it is described as an administration tool for IPv4 and IPv6 packet filtering and NAT. You can imagine it as a table, where you can add and remove rules, that will apply on incoming and outgoing IP packets and trigger actions.

Iptables come with a number of built in chains, but you can also add your own ones. Each chain is a list of rules, which can match a set of packets. You can see the chains in your host, by running:

    iptables -L

If the host is fresh new - let's say a linux container just created for test purposes - the result should be something like this:

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
```

Here you can see three chains: *input*, *forward* and *output*. The first and the last contain rules that apply to packets coming in and going out of the host. *Forward* will apply on packets that go through the host. On the command outcome above, we see that no chain contains any rules.

In order to test how you can block traffic with iptables you can run:

```
apt install apache2
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -j DROP
```

The apt command assumes an Ubuntu installation and just installs an Apache web server, that will expose an HTML page in port 80. The first iptables rule is added in the *Input* chain and accepts tcp packets, through the port 80. The last rule drops all packets that arrive in the *Input* chain. Since rule order is considered, HTTP packets though 80 will pass from the firewall, since the rule that accepts them precedes the dropping rule.

Now the *Input* chain is like this:

```
root@iptables:~# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
DROP       all  --  anywhere             anywhere  
```

If you type the IP address of the testing host in a browser, that has access to the host subnet, you will be able to see Apache content. On the contrary, if you *ping* the host, you will receive no answer, since the *ping* packets will never pass iptables.

In order to get rid of the rule that drops all IP traffic you can:

```
iptables -L --line-numbers
iptables -D ${rule to be deleted}
```

The first command will show the commands and their id numbers, necessasry for the second, deleting command. Now, you can ping the testing host as well.

> TODO port forwarding
