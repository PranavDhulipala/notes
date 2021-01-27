# notes

Connect container directly to a local network using ipvlan. Use ipvlan only if there's a wireless interface available on the device otherwise always use macvlan as macvlans are not built to work on wireless interfaces(refer https://www.kernel.org/doc/Documentation/networking/ipvlan.txt and http://hicu.be/macvlan-vs-ipvlan)

Macvlan and Ipvlan cannot work together.

Enable ipvlan feature in docker by making changes to /etc/docker/daemon.json (this might be different depending on your device, I tested this on a jetson nano).

```
{
    "experimental": true,
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```

Create a docker network interface using ipvlan using:

```
sudo docker network create -d ipvlan -o ipvlan_mode=l2 -o parent=wlan0 --subnet 192.168.0.0/24 --gateway 192.168.0.1 --ip-range 192.168.0.32/27 --aux-address 'host=192.168.0.63' <name>
```

parent can be eth0 or wlan0 depending on the network interface
--subnet flag creates a non-overlapping subnetwork for the network which is not a subdivision of an existing network. It is purely for ip-addressing purposes.
--gateway flag selects the ipv4 or ipv6 gateway for the master subnet.
--ip-range assigns the ip to the container from a sub-range. In the above case the ipranges from 192.168.0.32 to 192.168.0.63 are blocked for the containers.
--aux-adress reserves an address from our network range for use by the host interface. In the above case the last ip from the iprange i.e. 192.168.0.63 is used to direct the network packets.
name is a string for your interface

check the networks created by docker using: 

```
docker network ls
```

Now to create a new ipvlan interface on the host using the wlan0 network interface (assuming the devices will be connected to the internet using wifi, else use macvlan).

The idea here is to create a ipvlan interface to route traffic through the blocked ip on for the docker bridge 

```
sudo ip link add link wlan0 <name> type ipvlan mode l2
```
The above command creates a ipvlan interface on the host device  using l2 mode(The difference between L2 and L3 is that L2 allows the virtual interface to be bridged with the physical interface, which means it can have an address in the same subnet as the physical interface and L2 traffic is correctly relayed. L3 mode instead does not relay L2 traffic, and requires configuration as an IPv4 router: different subnets, need to setup routes, and so on).

```
sudo ip addr add 192.168.0.63/32 dev <name>
```

The above ip address is added to the ipvlan acting like a gateway and is the same one reserved using the aux address in the docker network setup.

```
sudo ip link set dev <name> up
```

The above command sets up a link to the ipvlan interface namely ipv0 created using the previous command.

```
sudo ip route add 192.168.0.32/27 dev <name>
```

The above command routes the ip ranges from 192.168.0.32 to 192.168.0.63 through 192.168.0.63 

Note:

Use the following command to delete the ipvlan interface added to the device

```
sudo ip link delete <name>
```

Use the following command to delete a network from docker

```
docker network rm <name>
```
Test the above setup by building and running a container in docker

```
docker build -t <container_name> .
```
replace the <name> in the below command with the name of your ipvlan docker network which was created previously. 
    
```
docker run --net=<name> --ip=192.168.0.32 -it --rm <container_name>
```
