# notes

Connect container directly to a local network using ipvlan. Use ipvlan only if there's a wireless interface available on the device otherwise always use macvlan as macvlans are not built to work on wireless interfaces(refer https://www.kernel.org/doc/Documentation/networking/ipvlan.txt and http://hicu.be/macvlan-vs-ipvlan)

Macvlan and Ipvlan cannot work together.

Enable ipvlan feature in docker by making changes to /etc/docker/daemon.json
