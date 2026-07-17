<h1>Enterprise Campus Network - VLAN Segmentation, EtherChannel & Inter-VLAN Routing</h1>

<p>
<b>Tool:</b> Cisco Packet Tracer &nbsp;|&nbsp; <b>Status:</b> Phase 1 Complete (Core Switching & Routing) &nbsp;|&nbsp; <b>Author:</b> Kamsi
</p>

<h2>Description</h2>
<p>
This project is a simulated small-to-medium enterprise network built in Cisco Packet Tracer. It models a
realistic three-tier design: an ISP edge router providing WAN/Internet connectivity, a Layer 3 distribution
switch handling inter-VLAN routing, and two Layer 2 access switches distributing traffic to four
departmental VLANs. Redundant links between the access and distribution layers are bundled with
EtherChannel and protected from loops with RSTP.
</p>
<p>
This is the first phase of a larger build. I'm documenting and pushing this to GitHub as I go so the
history reflects how I actually work through a design: plan the addressing scheme, build the topology,
configure it layer by layer, break it, and fix it. Planned Phase 2 additions include OSPF, extended ACLs,
SSH/Telnet remote management, and FTP.
</p>
<br />

<h2>Topology</h2>
<p align="center">
<img src="https://imgur.com/ZiGfN4E" height="85%" width="85%" alt="Network Topology Diagram"/>
</p>

<h2>Network Summary</h2>

<table>
<tr><th>Device</th><th>Model</th><th>Role</th></tr>
<tr><td>ISP-R1</td><td>Cisco 2911</td><td>Simulated ISP / Internet edge</td></tr>
<tr><td>R1</td><td>Cisco 2911</td><td>Edge router - static default route to ISP</td></tr>
<tr><td>DSW1</td><td>Cisco IE-3400</td><td>Layer 3 distribution switch - inter-VLAN routing (SVIs), EtherChannel to both access switches</td></tr>
<tr><td>ASW1</td><td>Cisco 2960-24TT</td><td>Access switch - VLAN 10 (Users) & VLAN 40 (IT)</td></tr>
<tr><td>ASW2</td><td>Cisco 2960-24TT</td><td>Access switch - VLAN 20 (Staff) & VLAN 30 (Management)</td></tr>
</table>

<h3>VLAN & Addressing Plan</h3>

<table>
<tr><th>VLAN ID</th><th>Name</th><th>Subnet</th><th>Access Switch</th><th>Notes</th></tr>
<tr><td>10</td><td>USERS</td><td>192.168.10.0/24</td><td>ASW1</td><td>General end-user workstations</td></tr>
<tr><td>20</td><td>STAFF</td><td>192.168.20.0/24</td><td>ASW2</td><td>Staff workstations</td></tr>
<tr><td>30</td><td>MANAGEMENT</td><td>192.168.30.0/24</td><td>ASW2</td><td>Management-tier PCs</td></tr>
<tr><td>40</td><td>IT</td><td>192.168.40.0/24</td><td>ASW1</td><td>IT workstations + SVR1 (DHCP/DNS server)</td></tr>
<tr><td>-</td><td>WAN Link (R1 - DSW1)</td><td>10.0.0.0/30</td><td>-</td><td>Point-to-point routed link</td></tr>
</table>

<h3>EtherChannel Bundles</h3>

<table>
<tr><th>Bundle</th><th>Members</th><th>Protocol</th><th>Load-Balancing Method</th></tr>
<tr><td>DSW1 &lt;-&gt; ASW1</td><td>DSW1 Gi1/3, Gi1/4 &lt;-&gt; ASW1 Gi0/1, Gi0/2</td><td>LACP</td><td>src-dst-ip</td></tr>
<tr><td>DSW1 &lt;-&gt; ASW2</td><td>DSW1 Gi1/5, Gi1/6 &lt;-&gt; ASW2 Gi0/1, Gi0/2</td><td>LACP</td><td>src-dst-ip</td></tr>
<tr><td>ASW1 &lt;-&gt; ASW2</td><td>Fa0/23, Fa0/24 (both switches)</td><td>LACP</td><td>src-dst-mac</td></tr>
</table>

<br />

<h2>Technologies & Concepts Implemented</h2>

- <b>VLANs & 802.1Q Trunking</b> - four departmental VLANs with tagged trunk links between switches
- <b>Inter-VLAN Routing</b> - Layer 3 SVIs on DSW1 (router-on-a-stick avoided in favor of a routed distribution switch)
- <b>DHCP</b> - centralized DHCP services scoped per VLAN
- <b>DNS</b> - local name resolution for internal hosts
- <b>RSTP (Rapid Spanning Tree)</b> - loop prevention across redundant access/distribution links, with PortFast and BPDU Guard on access ports
- <b>EtherChannel (LACP)</b> - link aggregation and load balancing between distribution and access layers
- <b>Static Routing</b> - default route from R1 out to the simulated ISP
- <b>Redundant Link Design</b> - dual uplinks between every switch pair for failover
<br />

<h2>Planned for Phase 2</h2>

- <b>OSPF</b> - replace/supplement static routing with dynamic routing
- <b>Extended ACLs</b> - inter-VLAN traffic filtering (e.g., restricting USERS from reaching IT/MANAGEMENT)
- <b>SSH</b> - disable Telnet in favor of encrypted remote management
- <b>FTP</b> - internal file transfer service
<br />

<h2>Environment Used</h2>

- <b>Cisco Packet Tracer</b> 8.2.x
<br />

<h2>Build Walkthrough</h2>

<p align="center">

Base topology and cabling: <br/>
<img src="images/01-topology.png" height="80%" width="80%" alt="Base topology"/>
<br /><br />

VLAN creation and trunk configuration on access switches: <br/>
<img src="images/02-vlans-trunks.png" height="80%" width="80%" alt="VLAN and trunk config"/>
<br /><br />

EtherChannel (LACP) configuration and verification - <code>show etherchannel summary</code>: <br/>
<img src="images/03-etherchannel.png" height="80%" width="80%" alt="EtherChannel verification"/>
<br /><br />

Inter-VLAN routing via SVIs on DSW1 - <code>show ip route</code>: <br/>
<img src="images/04-inter-vlan-routing.png" height="80%" width="80%" alt="Inter-VLAN routing"/>
<br /><br />

RSTP verification with PortFast/BPDU Guard on access ports - <code>show spanning-tree</code>: <br/>
<img src="images/05-rstp.png" height="80%" width="80%" alt="RSTP verification"/>
<br /><br />

DHCP scopes and lease verification from client PCs: <br/>
<img src="images/06-dhcp.png" height="80%" width="80%" alt="DHCP verification"/>
<br /><br />

End-to-end connectivity test across VLANs (ping/traceroute): <br/>
<img src="images/07-connectivity-test.png" height="80%" width="80%" alt="Connectivity testing"/>

</p>

<h2>Troubleshooting Notes</h2>
<p>
Documenting issues I hit and fixed is part of the point of this repo - it shows the actual
troubleshooting process, not just a finished config.
</p>

- Trunk allowed-VLAN lists on ASW1/ASW2 initially omitted VLAN 40, which silently blocked IT traffic across the trunk despite the VLAN existing locally.
- Mismatched port-channel vs. physical interface configuration caused an EtherChannel to sit in <code>SD</code> (suspended) state; fixed by ensuring identical speed/duplex/switchport mode on all bundle members before enabling LACP.
- Inconsistent STP link-type settings on point-to-point EtherChannel links delayed convergence; explicitly setting link-type as point-to-point resolved it.
- Applied and tested an extended ACL to confirm VLAN 10 could not reach VLAN 40, then removed it since ACLs are formally scheduled for Phase 2.
<br />

<h2>What This Project Demonstrates</h2>
<p>
Built to reflect the kind of hands-on troubleshooting and configuration work expected in an entry-level
IT Support or Network Technician role: structured VLAN design, Layer 2/Layer 3 troubleshooting using
<code>show</code> commands, redundant link design with EtherChannel and RSTP, and clear technical
documentation - a skill that matters just as much as the config itself in a support/NOC environment.
</p>

<!--
Notes for filling this in before publishing:
- Replace /images/ placeholders with real exported screenshots and topology PNG from Packet Tracer.
- Consider adding a "Configs" folder with the raw running-config .txt for each device (sanitized of any personal info).
- Consider linking a short screen-recording walkthrough (YouTube/Loom) once Phase 2 is done.
-->
