# Seed Labs Guide
## LOGBOOK for the sniffing and spoofing guide


### Setup
We will be using 3 containers to run this lab, since we need 3 machines in this guide. These will be connected to the same LAN.

To build and run a container, we can do:
```bash
dcbuild
dcup
```

We will need to insert code inside the attacker container. We can easily do this using the `volumes/` folder in the VM, that is shared between the VM and the container.
The following entry in the dockerfile handles this:
```
volumes:
- ./volumes:/volumes
```
Since containers have a limited view of the network traffic, we need to use `host mode` for the attacker container. The following entry is necessary to allow the it to see all traffics.
```
network_mode: host
```
This makes the container have the same IP address as the host VM, putting it in the same network space as the VM.

To get the network interface for the VM, we can run:
```bash
ifconfig
```
We know that the ip address assigned to the VM is `10.9.0.1`, so we can look for that in the results.
We find this:

![Alt text](screenshots/w13/guide/ifconfig.png)

Since the network interface is `br-` concatenated with the network id, we could also run to find the network id:
```bash
docker network ls
```

Which would result in this:

![Alt text](screenshots/w13/guide/docker_network.png)

And the resulting network interface is: `br-9b97557cdc9c`.


### Using Scapy to Sniff and Spoof Packets
In the set of tasks for this lab, we'll be using Scapy for each task.

#### Task 1: Sniffing packets
We start by trying to understand the program that is given to us. As explained, this "sniffs" the packets from the given network interface, effectively printing information about each packet received in that interface. It is also filtering so that it only prints "ICMP" packets.

##### Task 1.A
We copied the program and ran it in the VM with sudo permissions by doing
```bash
sudo python3 sniff.py
```
We did not get any output. We had to ping the VM so that it received packets. We did this by getting inside one of the containers and running the following command inside that container
```bash
# Get inside the container
dockps
docksh 21949

# Inside the container
ping 10.9.0.1   # ping the VM
```

This resulted in the following output:

How we pinged the VM inside one of the containers:
![Alt text](screenshots/w13/guide/ping1a.png)

The output on the python program:
![Alt text](screenshots/w13/guide/sniff_packets_1a.png)

We can see both the request and the reply, as well as information about the packets.


Therefore, we can conclude that we were able to view the information about the packets received by the VM.

If we had simply ran:
```bash
python3 sniff.py
```
We would have gotten the following error:
![Alt text](screenshots/w13/guide/sniff_error.png)

This is basically saying that we do not have permissions to sniff packets. This is because we are not running the program with sudo/root permissions.

##### Task 1.B
We could see that the previous program was filtering for only ICMP packets. So, the result is already presented before.

We now want to capture any TCP packet that comes from a particular IP, let's say `10.9.0.5`, since this was the IP of the container that we previously used to ping the VM. The packets to capture must also have destionation port number 23.

We found information about how to build a Berkeley Packet Filter online. The resulting filter was `tcp and dst port 23 and src host 10.9.0.5` and we changed the program to use this filter.

<!--TODO-->
```python
pkt = sniff(iface='', filter='tcp and dst port 23 and src host', prn=print_pkt)
```

Since ping issues ICMP packets, we will not be able to do the same as in the previous task. We will need to use a different program to send TCP packets to the VM.

In this case, we will use `telnet`.

Firstly, we start the sniffing program by running
We can run the program with:
```bash
sudo python3 sniff.py
```

Then, we connect to the container:
```bash
# Enter the container
dockps
docksh 21949

# Inside the container
telnet 10.9.0.1     # connect to the VM with TCP
```
We also input the username and password for the VM, which are `seed` and `dees` respectively.

We can see the output of the telnet program here:
![Alt text](screenshots/w13/guide/telnet.png)

And the output of the sniffing program here:

The output is very large, but only a section are shown here:
![Alt text](screenshots/w13/guide/sniff_packets_1b.png)



