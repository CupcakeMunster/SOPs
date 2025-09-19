# SOPs
Juniper switches can be configured to send all captured traffic from a port analyzer (mirror) to a remote VLAN. This is helpful for when a technician is troubleshooting remotely but still needs to evaluate a raw packet capture. Since Junipers built in tcpdump tool can only show packets destined to the routing engine itself, this method is a good solution for obtaining complete visibility of network traffic. 

## Capturing Switch Setup
First, we create a VLAN to use as an isolated network for packet captures.
```
set vlans PORTMIRROR-MONITOR0 description "A VLAN for remote monitoring of switch interface traffic"
set vlans PORTMIRROR-MONITOR0 vlan-id 100
```

We'll create an analyzer service called "remote-analyzer". The ingress and egress traffic on interface ge-0/0/1.0 will be the input traffic to this analyzer.
The output of the analyzer will be the VLAN we just created.
```
set forwarding-options analyzer remote-analyzer input ingress interface ge-0/0/1.0
set forwarding-options analyzer remote-analyzer input egress interface ge-0/0/1.0
set forwarding-options analyzer remote-analyzer output vlan PORTMIRROR-MONITOR0
```

Once we set the trunk interface to allow our new VLAN, all traffic entering and leaving interface ge-0/0/1 will be copied and sent out on PORTMIRROR-MONITOR0 over the trunk port ae0. The capturing switch config is completed.
```
set interface ae0.0 family ethernet-switching vlan members PORTMIRROR-MONITOR0
```

## Transitory Switch Setup
Any switches that act as a device in between the capturing switch and remote switch need to have mac learning disabled on our new VLAN. This will likely be distribution, core, or spine devices as they often only carry transitory traffic.  

For EX series switches.
```
set vlans PORTMIRROR-MONITOR0 description "A VLAN for remote monitoring of switch interface traffic"
set vlans PORTMIRROR-MONITOR0 vlan-id 100
set vlans PORTMIRROR-MONITOR0 no-mac-learning
```
For QFX series switches.
```
set vlans PORTMIRROR-MONITOR0 description "A VLAN for remote monitoring of switch interface traffic"
set vlans PORTMIRROR-MONITOR0 vlan-id 100
set vlans PORTMIRROR-MONITOR0 switch-options no-mac-learning
```

And we'll add the new VLAN to any needed interfaces.
```
set interface ae1.0 family ethernet-switching vlan members PORTMIRROR-MONITOR0
set interface ae2.0 family ethernet-switching vlan members PORTMIRROR-MONITOR0
```

## Remote Switch Setup
On the remote switch, we'll create our VLAN.
```
set vlans PORTMIRROR-MONITOR0 description "A VLAN for remote monitoring of switch interface traffic"
set vlans PORTMIRROR-MONITOR0 vlan-id 100
```

We create our analyzer again, but the input and output are reversed. This input is just the ingress of the PORTMIRROR-MONITOR0 vlan. The output is whatever interface is being used for packet capture with a device tool like wireshark or tcpdump.
```
set forwarding-options analyzer remote-analyzer input ingress vlan PORTMIRROR-MONITOR0
set forwarding-options analyzer remote-analyzer output interface ge-1/0/22.0
```

The last step is to configure the remote capture interface as a trunk and to add our VLAN to all necessary interfaces.
```
set interfaces ge-1/0/22 unit 0 family ethernet-switching interface-mode trunk
set interfaces ge-1/0/22 unit 0 family ethernet-switching vlan members PORTMIRROR-MONITOR0
set interfaces ae0 unit 0 family ethernet-switching vlan members PORTMIRROR-MONITOR0
```
