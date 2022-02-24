[<<](../README.md)

# Creating a Linux Bridge to Connect the Container

Yet on the host let's create the network devices for our container.

First let's save the process ID of our container that is running under the unshare command for the moment to an environment variable.

```
export CPID=$(pidof unshare)
```

Creating the bridge:
```
sudo ip link add br0 type bridge
```

Setting up a layer 3 interface with an IP address to work with this bridge:
```
sudo ip addr add dev br0 10.0.0.1/16
sudo ip link set br0 up
```

Creating a Veth pair one to be connected to the bridge and the other to be connected to the container:
```
sudo ip link add name h$CPID type veth peer name c${CPID}
```

Now let's move one of those connection ends to the new net namespace for our container to be able to see that connection.
```
sudo ip link set c${CPID} netns ${CPID}
```

Don't forget to bring up the host end of this veth pair. Notice that it's down at this point:
```
[vagrant@container-host ~]$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:37:73:02 brd ff:ff:ff:ff:ff:ff
3: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 6a:b6:0c:c4:ea:f2 brd ff:ff:ff:ff:ff:ff
5: h5536@if4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether e6:91:07:04:f6:40 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

```
sudo ip link set h${CPID} up
```
And attach that connection to the bridge we've created:

```
sudo ip link set h$CPID master br0 up
```

Finally let's get back to our container on the other terminal and configure basic networking inside the container. Take a look with the `ip link` command that we have a loopback in DOWN state and a non configured ethernet interface also in DOWN state now.

```
[root@littlebox littlebox]# ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
4: c5536@if5: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether 52:23:95:68:24:6b brd ff:ff:ff:ff:ff:ff

```

Let's put the loopback up:
```
ip link set lo up
```

Let's use that veth interface (use the name you got from your system with the ip link command) and add an ip address on the same network where our bridge is:
```
ip addr add dev c${CPID} 10.0.0.2/16
ip link set c${CPID} up
```

We still need to configure at least a default route to be able to reach outside:
```
ip route add default via 10.0.0.1
```

Finally from our container terminal we can ping an address that is outside of that network namespace that we're using.

```
[root@littlebox littlebox]# ping 10.0.0.1
PING 10.0.0.1 (10.0.0.1): 56 data bytes
64 bytes from 10.0.0.1: seq=0 ttl=64 time=0.151 ms
64 bytes from 10.0.0.1: seq=1 ttl=64 time=0.127 ms
```
---
But what if we want to make it connect to the internet?

In order to provide internet access to our container we need to do at the bare minimum 2 things: allow traffic forwarding from system's internal resources to the outside and perform NAT operations on behalf of those resources on the host give them an external IP address that can actually talk to the internet and receive the traffic back.

Only 2 commands are needed for the testing scenario here.

First we change the forward policy for the entire system:

```
sysctl -w net.ipv4.ip_forward=1
```
OBS: this above is a non permanent solution to allow forwarding to happen on the host VM. If you want it to be permanent you need to edit the /etc/sysctl.conf on the line `net.ipv4.ip_forward = 0` and then reload it with `sysctl -p /etc/sysctl.conf`.


Then we create an iptables rule that will NAT all traffic coming from the container network to the outside network. It uses the nat table, runs on the POSROUTING chain (after it decided where to send the packet to) when the packets are going to host's exit interface and masquarades the source address, i. e., translates it to that exit interface IP therefore allowing it to go out and come back.

```
 iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE 
```
After that we can ping an external address if your VM has internet access:
```
[root@littlebox littlebox]# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=55 time=10.631 ms
64 bytes from 8.8.8.8: seq=1 ttl=55 time=12.105 ms
64 bytes from 8.8.8.8: seq=2 ttl=55 time=13.251 ms
64 bytes from 8.8.8.8: seq=3 ttl=55 time=11.950 ms
```

Hey, wait a moment! What about DNS? Can we have name resolution on that particular namespace? And the answer is yes, sure. Just do the basics for now and use an external public DNS server:
```
echo 'nameserver 8.8.8.8' >> /etc/resolv.conf
```

And boom! We have something like this:
```
[root@littlebox littlebox]# ping www.redhat.com
PING www.redhat.com (23.6.82.130): 56 data bytes
64 bytes from 23.6.82.130: seq=0 ttl=57 time=16.536 ms
64 bytes from 23.6.82.130: seq=1 ttl=57 time=23.863 ms
64 bytes from 23.6.82.130: seq=2 ttl=57 time=17.036 ms
```

