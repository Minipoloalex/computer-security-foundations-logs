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
