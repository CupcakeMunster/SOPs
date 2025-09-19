# SOPs
Juniper switches can be configured to send all captured traffic from a port analyzer (mirror) to a remote VLAN. This is helpful for when a technician is troublshooting remotely but still needs to evaluate a raw packet capture. Since Junipers built in tcpdump tool can only show packets destined to the routing engine itself, this method is a good solution for obtaining complete visability of network traffic.

First, we create a VLAN to use as an isolated network for packet captures.
```
set vlans PORTMIRROR-MONITOR0 description "A VLAN for remote monitoring of switch interface traffic"
set vlans PORTMIRROR-MONITOR0 vlan-id 100
```

We'll create an analyzer service called "myfirstanalyzer". The ingress and egress traffic on interface ge-0/0/1.0 will be the input traffic to this analyzer.
The output of the analyzer will be the VLAN we just created.
```
set forwarding-options analyzer myfirstanalyzer input ingress interface ge-0/0/1.0
set forwarding-options analyzer test input egress interface ge-0/0/1.0
set forwarding-options analyzer test output vlan PORTMIRROR-MONITOR0
```

Once we set the trunk interface to allow our new VLAN, all traffic entering and leaving interface ge-0/0/1 will be copied and sent out on PORTMIRROR-MONITOR0 over the trunk port ae0. The capturing switch config is completed.
```
set interface ae0.0 family ethernet-switching vlan members PORTMIRROR-MONITOR0
```
