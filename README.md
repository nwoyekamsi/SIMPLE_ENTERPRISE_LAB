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
<img src="https://github.com/user-attachments/assets/3256bdcf-9ff2-45c1-baed-309d528fb749" height="85%" width="85%" alt="Network Topology Diagram"/>
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
- <b>STP VLAN Load Balancing</b> - reduce congestion on the single distribution switch
- <b>Redundant Distribution Layer</b> - Backup distribution switch
- <b>WAN/Internet Redundancy</b> - Deploy a secondary router connected to the ISP
<br />

<h2>Environment Used</h2>

- <b>Cisco Packet Tracer</b> 
<br />

<h2>Build Walkthrough</h2>

<p align="center">

Base topology and cabling: <br/>
<img src="https://github.com/user-attachments/assets/7e11519d-03aa-42fc-996f-c2227ad36d53" height="50%" width="50%" alt="Base topology"/>
<br /><br />

VLAN creation and trunk configuration on access switches: <br/>
<img src="https://github.com/user-attachments/assets/2968c3c0-56ec-45f8-92f3-5f797a4f503e" height="50%" width="50%" alt="VLAN and trunk config"/>
<img src="https://github.com/user-attachments/assets/d0855212-c596-4349-81fc-b31a4381fe1b" height="50%" width="50%" alt="VLAN and trunk config"/>
<img src="https://github.com/user-attachments/assets/7f3245fe-5f0b-4f0c-b750-76b1eb51272b" height="50%" width="50%" alt="VLAN and trunk config"/>
<br /><br />

EtherChannel (LACP) configuration and verification - <code>show etherchannel summary</code>: <br/>
<img src="https://github.com/user-attachments/assets/a6ac6bac-00f8-4b82-b9df-ae578e5f91d9" height="50%" width="50%" alt="EtherChannel verification"/>
<img src="https://github.com/user-attachments/assets/09e8aad7-dd0a-4d3c-9321-9dda9badd64f" height="50%" width="50%" alt="EtherChannel verification"/>
<img src="https://github.com/user-attachments/assets/f5dd23ca-59a8-48e6-9a36-f9a80a344b75" height="50%" width="50%" alt="EtherChannel verification"/>
<br /><br />

Inter-VLAN routing via SVIs on DSW1 - <code>show ip route</code>: <br/>
<img src="https://github.com/user-attachments/assets/e8bffec4-317a-42c1-8f81-f72348ab1bd5" height="50%" width="50%" alt="Inter-VLAN routing"/>
<br /><br />

RSTP verification with PortFast/BPDU Guard on access ports - <code>show spanning-tree summary</code>: <br/>
<img src="https://github.com/user-attachments/assets/68d0180c-3203-4695-a061-84e0e1321a9f" height="50%" width="50%" alt="RSTP verification"/>
<img src="https://github.com/user-attachments/assets/c50f3f48-2ba1-4179-bc8b-d2d5e8c2aeb5" height="50%" width="50%" alt="RSTP verification"/>
<img src="https://github.com/user-attachments/assets/8ed64406-01e2-4f11-9c66-7180904abfa8" height="50%" width="50%" alt="RSTP verification"/>
<br /><br />

DHCP scopes and lease verification from client PCs: <br/>
<img src="https://github.com/user-attachments/assets/548db6f0-8fbb-4d33-9008-0cf6bbb4282c" height="50%" width="50%" alt="DHCP verification"/>
<img src="https://github.com/user-attachments/assets/4937d4fc-1562-4ef1-9f47-3aee63064e13" height="50%" width="50%" alt="DHCP verification"/>
<img src="https://github.com/user-attachments/assets/e6584d7c-907a-4994-8ba7-d411c303c06d" height="50%" width="50%" alt="DHCP verification"/>
<br /><br />

End-to-end connectivity test across VLANs (ping/traceroute): <br/>
<img src="https://github.com/user-attachments/assets/397718df-2700-42be-b655-50cf5ec818b5" height="50%" width="50%" alt="Connectivity testing"/>
<img src="https://github.com/user-attachments/assets/1021cb85-113f-4a07-b49c-0b793f1a4413" height="50%" width="50%" alt="Connectivity testing"/>

</p>

<h2>Troubleshooting Notes</h2>
<p>
Documenting issues I hit and fixed is part of the point of this repo - it shows the actual
troubleshooting process, not just a finished config.
</p>

- <b>EtherChannel forming in SU (Layer 2, not bundled) state despite correct LACP configuration -</b> Interface-level configuration matched on both ends, so I checked the physical layer and found a media mismatch — one member link was cabled with FastEthernet (100 Mbps) while its partner was GigabitEthernet (1000 Mbps). EtherChannel requires all bundle members to match on speed and duplex, so the mismatched cable was replaced with GigabitEthernet on both ends rather than forcing the faster port down to 100 Mbps, preserving full bundle bandwidth.
- <b>DNS name resolution failing (ping by hostname unsuccessful) despite a correctly configured DNS server address on the DHCP scope -</b> Verified the DHCP server configuration first and confirmed the DNS server IP was scoped correctly. The root cause was on the client side: the test PC held an existing DHCP lease issued before the DNS server setting was added, so it hadn't yet received the updated option. Releasing and renewing the IP configuration (ipconfig /renew equivalent in Packet Tracer) pulled the current lease, including the DNS server address, and resolved the failure — a reminder that DHCP option changes don't retroactively apply to already-leased clients. <b>**NOTE: THE PCs WITHOUT LABELED IP ADDRESSES ARE DYNAMICALLY CONFIGURED.**</b>
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
