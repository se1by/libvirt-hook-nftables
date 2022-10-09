QemuNFTables
===

Provide port forwarding for libvirt via nftables.

How it works
===

The script creates a ``nat`` table, as well as a ``prerouting``, ``postrouting`` and ``output`` chain with relevant hooks and
priorities.

After that, it creates a SNAT rule for the guest IPs /24 and a DNAT rule which forwards the port. As there is no
limitation regarding the device or IP, it also works if a vm (or the host itself) tries to access a service hosted in another vm by the
public ip (e.g. you run git.example.com and forward requests to port 22 to vm2, vm1 can still access that by the public
IP).

mapping.json
===
This script expects a ``mapping.json`` file in the ``/etc/libvirt/hooks`` directory. It should look follow this format:

```json
{
  "testvm1": { // name of your VM
    "guest_ip": "192.168.122.2", // private ip of your vm
    "host_ip": "10.0.0.1", // public ip of your host
    "ports": [
      {
        "host": 30001, // port on the host.
        "guest": 30002, // port on the vm. If omitted, the host port is used
        "protocol": "tcp" // tcp or udp
      }
    ]
  }
}

```
Note that comments are not allowed in json, so remove them if you copied this example.

Installation
===
Just copy the ``qemu`` file to ``/etc/libvirt/hooks/qemu`` (note it's a file, not a directory!) 
and create a ``mapping.json``. 

Caveats
===
This script doesn't completely cleanup after itself: The SNAT rule and the table/chains won't get removed automatically.
Run ``nft delete table nat`` to do so manually.

# !IMPORTANT!

This script assumes that the used libvirt network uses mode ``open``. If it doesn't, libvirts own IPTables rules will
likely interfere. If you have no special needs, you can copy the default network xml and replace the forward mode
``nat`` with ``open``. After that undefine the original ``default`` network and create yours (and probably mark it as 
autostart).