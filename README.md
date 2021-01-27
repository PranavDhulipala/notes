# notes

Connect container directly to a local network using ipvlan. Use ipvlan only if there's a wireless interface available on the device otherwise always use macvlan as macvlans are not built to work on wireless interfaces(refer https://www.kernel.org/doc/Documentation/networking/ipvlan.txt and http://hicu.be/macvlan-vs-ipvlan)

Macvlan and Ipvlan cannot work together.

Enable ipvlan feature in docker by making changes to /etc/docker/daemon.json

{
    "experimental": true,
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}

Create a docker network interface using ipvlan using:

sudo docker network create -d ipvlan -o ipvlan_mode=l2 -o parent=wlan0 --subnet 192.168.0.0/24 --gateway 192.168.0.1 --ip-range 192.168.0.30/27 --aux-address 'host=192.168.0.25' ipvlan210

--subnet flag creates a non-overlapping subnetwork for the network which is not a subdivision of an existing network. It is purely for ip-addressing purposes.
--gateway flag selects the ipv4 or ipv6 gateway for the master subnet.
--ip-range assigns the ip to the container from a sub-range.
--aux-adress reserves an address from our network range for use by the host interface.

