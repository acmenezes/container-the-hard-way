# Creating a Linux Bridge to Connect the Container

Now is the time to use our bridge-utils package that we have installed at the beginning. The most important command suite is brctl.

```
sudo ip link add br0 type bridge
sudo ip addr add dev br0 10.0.0.1/16
sudo ip link set br0 up
```

```
sudo ip link add name h$CPID type veth peer name c${CPID}
sudo ip link set c${CPID} netns ${CPID}
```

```
ip link set lo up
ip link addr add dev <CPID> 10.0.0.2/24
ip route add default via 10.0.0.1
```

```
ping 10.0.0.1
ping 192.168.121.30
```

Now to get to the internet we need to set up IPv4 forwarding and NAT using iptables. Let's do it!

