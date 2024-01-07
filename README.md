# docker-network-inspecting

## `Docker Status`

Making sure Docker is running in the background. So, let's check the Docker status.

```bash
sudo systemctl status docker
```

![docker-status](https://lab-bucket.s3.brilliant.com.bd/labthumbnail/85711c14-03fb-4c03-8e71-ab800a8f5a4e.png)


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

## `Inspecting Bridge Network`

```bash
docker network inspect bridge
```

```bash
[
    {
        "Name": "bridge",
        "Id": "5bd6001154b40902583da5d50908a7226b82b809d4af39949cdda897bc1a3170",
        "Created": "2023-11-30T06:48:14.189396785Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
...
...
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```


Let's talk about `subnet` and `gateway` under the "Config" section and some other configurations under the "Options" section.

## `Config`
This configuration indicates that the Docker bridge network (`docker0`) is configured with a subnet of `172.17.0.0/16`, and the default `gateway` for containers on this network is set to `172.17.0.1`. Containers connected to this bridge network will be assigned IP addresses within the specified subnet range, and the bridge interface (`docker0`) will act as the gateway for their outbound traffic.

## `Options`
The Docker bridge network `docker0` is configured with specific options:
- `Default Bridge`: Marked as the default bridge network.
- `Inter-Container Communication (ICC)`: Enabled for communication between containers on the same bridge.
- `IP Masquerade`: Allows NAT for outbound packets from containers.
- `Host Binding IPv4`: Binds the bridge network to all available host network interfaces.
- `Bridge Name`: Identified as "docker0" for the Docker bridge network.
- `MTU (Maximum Transmission Unit)`: Configured with an MTU of 1500 bytes for packet transmission.

---

## `Create a Bridge network with a Subnet and Gateway`

```bash
docker network create --subnet 10.1.0.0/16 --gateway 10.1.0.1 \
--driver=bridge --label=development v-br
```
***Expected Output***

```bash
{
        "Name": "v-br",
        "Id": "cb3d7ed948f1becf542b540743b8adc702b624e5042bd8073496a7325558d790",
        "Created": "2024-01-07T13:24:50.67153161Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.1.0.0/16",
                    "Gateway": "10.1.0.1"
                }
            ]
        }
```

## `Run a Docker Container Under a Specific Network`

Let's verify the newly created `v-br` bridge network. The idea is that we are going to create a container inside the `v-br` network and inspect the assigned IP address, whether it is from the `v-br` subnet or not, and also check the default gateway.

So first of all, we need to run a container. Here, we will run Ubuntu as a container.

```bash
docker run  --name host-vm -it --network v-br ubuntu /bin/bash
```
## `Prerequisites`

We need to install some prerequisites to use some IP utilities.

```bash
apt update
apt install iproute2
apt install net-tools
```
Now let's find out the network interface of this container.

```bash
ifconfig
```

***Expected Output***
```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.0.2  netmask 255.255.0.0  broadcast 10.1.255.255
        ether 02:42:0a:01:00:02  txqueuelen 0  (Ethernet)
        RX packets 2296  bytes 30753167 (30.7 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1403  bytes 104325 (104.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Let's check the default gateway.

```bash
netstat -rn
```

```bash
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.1.0.1        0.0.0.0         UG        0 0          0 eth0
10.1.0.0        0.0.0.0         255.255.0.0     U         0 0          0 eth0
```

Bingo! Everything is matched according to the `v-br` bridge network.
