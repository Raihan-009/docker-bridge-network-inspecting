# docker-network-inspecting

## `Docker Status`

Making sure Docker is running in the background. So, let's check the Docker status.

```bash
sudo systemctl status docker
```

![docker-status](https://lab-bucket.s3.brilliant.com.bd/labthumbnail/1fe9dc4c-5ff9-4c98-812e-fa8d4fcf27e0.png)


## `Available Network Interfaces`

Let's execute `ifconfig` to get all network devices available on our host machine.

```bash
ifconfig
```
Available interfaces
```bash
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:f1ff:fe36:8f4d  prefixlen 64  scopeid 0x20<link>
        ether 02:42:f1:36:8f:4d  txqueuelen 0  (Ethernet)
        RX packets 9566  bytes 666436 (666.4 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 13572  bytes 112793430 (112.7 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.0.0.25  netmask 255.255.255.0  broadcast 10.0.0.255
        inet6 fe80::f816:3eff:fec8:b89c  prefixlen 64  scopeid 0x20<link>
        ether fa:16:3e:c8:b8:9c  txqueuelen 1000  (Ethernet)
        RX packets 2331178  bytes 1547568823 (1.5 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2451074  bytes 477520472 (477.5 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 11558  bytes 1441775 (1.4 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 11558  bytes 1441775 (1.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Here, the `docker0` interface is a virtual Ethernet bridge that Docker uses for communication between containers on the same host. Containers connected to this bridge can communicate with each other using IP addresses in the 172.17.0.0/16 range.

The `ens3` interface is an Ethernet interface with an assigned IP address connected to the host machine's physical network, providing external network connectivity.

The `lo` interface is the loopback interface, used for local communication within the host. It always has the IP address 127.0.0.1 and is commonly used for applications to communicate with themselves.

## `Available Networking Commands`

```bash
docker network --help
```

![available-commands](https://lab-bucket.s3.brilliant.com.bd/labthumbnail/ba99f1bc-1b10-4e82-a800-441076cb559e.png)

## `Listing Network`
```bash
docker network ls
```
```
NETWORK ID     NAME         DRIVER    SCOPE
5bd6001154b4   bridge       bridge    local
33b130858694   host         host      local
667c9b003ef2   none         null      local
```

This command provides an overview of the Docker networks present on our host, including their names, network IDs, drivers, and scopes.