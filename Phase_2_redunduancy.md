<h1>Phase 2: Redundant Distribution & Edge Layer with HSRP + OSPF</h1>

<p>
<b>Tool:</b> Cisco Packet Tracer &nbsp;|&nbsp; <b>Status:</b> Phase 2 Complete (Redundancy & Dynamic Routing) &nbsp;|&nbsp; <b>Author:</b> Kamsi
</p>
<p>
<b>Builds on:</b> <a href="./README.md">Phase 1 - VLAN Segmentation, EtherChannel & Inter-VLAN Routing</a>
</p>

<h2>Description</h2>
<p>
Phase 1 proved the base design worked, but it had two single points of failure: one distribution switch
and one edge router. If either went down, the entire network — or the entire path to the Internet —
went down with it. Phase 2 rebuilds those two layers around redundancy instead of just connectivity.
</p>
<p>
A second Layer 3 distribution switch (DSW2) was added alongside DSW1, and a second edge router (R2) was
added alongside R1, each with its own independent link to the simulated ISP router (ISPR1). HSRP now
provides gateway failover between the two distribution switches, and the single static uplink from
Phase 1 was replaced with four dedicated routed point-to-point links running OSPF, so traffic has
multiple dynamically-selected paths out of the network instead of one static path.
</p>
<p>
Everything from Phase 1 — VLANs, DHCP, DNS, EtherChannel, RSTP — carries over unchanged and isn't
re-documented here. This file covers only what's new: the redundant hardware, HSRP, the routed OSPF
core, DHCP relay (now required since client VLANs are reachable through either distribution switch),
and extended ACLs for inter-VLAN security.
</p>
<br />

<h2>Updated Topology</h2>
<p align="center">
<img width="1845" height="1053" alt="image" src="https://github.com/user-attachments/assets/b3efb8af-b143-4cde-b87d-d25cf9078b0b" />
</p>

<h2>New Technologies & Concepts Implemented</h2>

- <b>Redundant Distribution Layer</b> - second Layer 3 switch (DSW2) alongside DSW1, sharing OSPF and HSRP responsibilities
- <b>Redundant Edge Layer</b> - second edge router (R2) alongside R1, each with its own independent ISP-facing link
- <b>HSRP</b> - first-hop gateway redundancy between DSW1 and DSW2, one virtual IP per VLAN
- <b>Routed Point-to-Point Links</b> - four dedicated /30 Layer 3 links between the routers and distribution switches, replacing the single static uplink from Phase 1
- <b>OSPF (Area 0)</b> - dynamic routing across the four routed links, with equal-cost paths enabling active/active (ECMP) forwarding through both routers
- <b>Loopback Interfaces & Explicit OSPF Router IDs</b> - stable per-device identity for OSPF, independent of any single physical interface
- <b><code>default-information originate</code></b> - dynamic propagation of the Internet default route into OSPF, instead of manually configuring a static default route on every distribution switch
- <b>DHCP Relay (<code>ip helper-address</code>)</b> - forwarding DHCP broadcasts from client VLANs to the central DHCP server, now required with a dual-distribution-switch design
- <b>Extended ACLs</b> - inter-VLAN traffic restriction, applied identically on both distribution switches so the policy holds regardless of which one is the active HSRP gateway
- <b>Failure-scenario testing</b> - deliberate device and link failures to verify the redundancy mechanisms actually behave as designed, not just that they're configured
<br />

<h2>Updated Architecture</h2>

<pre>
                         INTERNET
                            |
                          ISPR1
                        /       \
                200.1.1.0/30   200.1.1.4/30
                    /               \
                  R1                 R2
                 /  \               /  \
          10.0.1.0/30 10.0.2.0/30 10.0.3.0/30 10.0.4.0/30
               /          \       /          \
             DSW1 ==================== DSW2
               |         (EtherChannel)   |
              ASW1                      ASW2
               |                          |
             VLANs 10/40              VLANs 20/30
</pre>

<table>
<tr><th>Device</th><th>Model</th><th>Role in Phase 2</th></tr>
<tr><td>ISPR1</td><td>Cisco 2911</td><td>Simulated ISP router - now serves both R1 and R2 as independent uplinks, still provides DNS</td></tr>
<tr><td>R1</td><td>Cisco 2911</td><td>Edge router - OSPF peer to DSW1 and DSW2, static default route to ISPR1, propagates default route into OSPF</td></tr>
<tr><td>R2</td><td>Cisco 2911</td><td>Second edge router - independent ISP-facing link, OSPF peer to both distribution switches, active/active with R1</td></tr>
<tr><td>DSW1</td><td>Cisco IE-3400</td><td>Layer 3 distribution switch - OSPF, HSRP active/standby per VLAN, ACL enforcement</td></tr>
<tr><td>DSW2</td><td>Cisco IE-3400</td><td>Second Layer 3 distribution switch - OSPF, HSRP peer to DSW1, ACL enforcement</td></tr>
</table>
<br />

<h2>Addressing - New Links</h2>

<table>
<tr><th>Link</th><th>Network</th><th>Type</th></tr>
<tr><td>R1 &lt;-&gt; DSW1</td><td>10.0.1.0/30</td><td>Routed P2P (OSPF)</td></tr>
<tr><td>R1 &lt;-&gt; DSW2</td><td>10.0.2.0/30</td><td>Routed P2P (OSPF)</td></tr>
<tr><td>R2 &lt;-&gt; DSW1</td><td>10.0.3.0/30</td><td>Routed P2P (OSPF)</td></tr>
<tr><td>R2 &lt;-&gt; DSW2</td><td>10.0.4.0/30</td><td>Routed P2P (OSPF)</td></tr>
<tr><td>R1 &lt;-&gt; ISPR1</td><td>200.1.1.0/30</td><td>Static default route</td></tr>
<tr><td>R2 &lt;-&gt; ISPR1</td><td>200.1.1.4/30</td><td>Static default route</td></tr>
</table>

<table>
<tr><th>Device</th><th>Loopback Address</th><th>Purpose</th></tr>
<tr><td>R1</td><td>1.1.1.1/32</td><td>OSPF Router ID</td></tr>
<tr><td>R2</td><td>2.2.2.2/32</td><td>OSPF Router ID</td></tr>
<tr><td>DSW1</td><td>11.11.11.11/32</td><td>OSPF Router ID</td></tr>
<tr><td>DSW2</td><td>22.22.22.22/32</td><td>OSPF Router ID</td></tr>
</table>

<h3>HSRP Virtual Gateway IPs (per VLAN)</h3>

<table>
<tr><th>VLAN</th><th>Subnet</th><th>HSRP Virtual IP</th><th>Active</th><th>Standby</th></tr>
<tr><td>10 - USERS</td><td>192.168.10.0/24</td><td>192.168.10.23</td><td>DSW1</td><td>DSW2</td></tr>
<tr><td>20 - STAFF</td><td>192.168.20.0/24</td><td>192.168.20.23</td><td>DSW2</td><td>DSW1</td></tr>
<tr><td>30 - MANAGEMENT</td><td>192.168.30.0/24</td><td>192.168.30.23</td><td>DSW2</td><td>DSW1</td></tr>
<tr><td>40 - IT</td><td>192.168.40.0/24</td><td>192.168.40.23</td><td>DSW1</td><td>DSW2</td></tr>
</table>
<p><i>Active roles split across both switches per VLAN so both distribution switches carry live traffic under normal conditions, rather than one switch sitting fully idle.</i></p>
<br />

<h2>HSRP - Gateway Redundancy</h2>
<p>
Each VLAN's default gateway is now a shared virtual IP between DSW1 and DSW2 instead of a single
physical switch address. If the active switch fails, the standby switch takes over the virtual IP
automatically — end devices never need to change their configured default gateway.
</p>

```cisco
interface Vlan10
 ip address 192.168.10.2 255.255.255.0
 standby 10 ip 192.168.10.23
 standby 10 priority 110
 standby 10 preempt
```
<br />

<h2>OSPF Design</h2>
<p>
The four router-to-distribution links run OSPF Area 0 with equal cost, allowing both R1 and R2 to
actively forward traffic (ECMP) instead of one router sitting idle as a cold standby.
</p>

```cisco
router ospf 1
 router-id 1.1.1.1
 network 10.0.1.0 0.0.0.3 area 0
 network 10.0.2.0 0.0.0.3 area 0
 default-information originate
```

<p>
<code>default-information originate</code> lets R1 and R2 inject the Internet default route into OSPF,
so DSW1 and DSW2 learn the route to the ISP dynamically instead of needing a manually maintained static
route on every distribution switch. If a router loses its ISP link, it stops advertising the default
route and traffic shifts to the surviving router automatically.
</p>
<br />

<h2>DHCP Relay</h2>
<p>
With client VLANs now reachable through either distribution switch, DHCP broadcasts need to be
explicitly forwarded to the central DHCP server rather than relying on a single local Layer 2 segment.
</p>

```cisco
interface Vlan10
 ip helper-address 192.168.40.100
```
<p>Configured identically on both DSW1 and DSW2 for every client VLAN.</p>
<br />

<h2>Extended ACL - Inter-VLAN Restriction</h2>
<p>
Applied identically on both DSW1 and DSW2 — since either can be the active HSRP gateway for VLAN 10 at
any given time — to block VLAN 10 from reaching the server VLAN while leaving all other inter-VLAN and
Internet traffic untouched.
</p>

```cisco
ip access-list extended 100
 deny icmp 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255
 permit ip any any
!
interface 10
 ip access-group 100 in
```
<br />

<h2>Design Iteration: Why HSRP Wasn't Used Between the Routers</h2>
<p>
This is the part of Phase 2 worth explaining in an interview, not just listing in a table.
</p>
<p>
HSRP was the correct fit for the first half of the redundancy problem: giving the internal network a
resilient default gateway between DSW1 and DSW2. The initial plan was to extend that same approach to
R1 and R2, using HSRP to provide a shared virtual gateway for outbound traffic to the Internet.
</p>
<p>
In practice, this added complexity without delivering reliable results. HSRP is designed to solve "which
device should be the default gateway for my hosts" — a Layer 2/first-hop problem. Applying it to the
routers for outbound path selection was asking it to solve a different problem: which of two available
paths should traffic take to leave the network. That's a routing decision, not a gateway decision, and
the desired failover behavior wasn't happening reliably.
</p>
<p>
The fix was recognizing the mismatch and switching to the protocol actually built for that job. OSPF was
configured across four dedicated routed <code>/30</code> links between the routers and distribution
switches, letting the network dynamically select and fail over between paths to the Internet. This
separated the two concerns that had gotten blurred together:
</p>

<table>
<tr><th>Protocol</th><th>Question it answers</th></tr>
<tr><td>HSRP</td><td>Which device is the default gateway for my hosts? (DSW1 &lt;-&gt; DSW2)</td></tr>
<tr><td>OSPF</td><td>Which Layer 3 path should traffic take to leave the network? (DSW1/DSW2 &lt;-&gt; R1/R2)</td></tr>
</table>

<p>
Once each protocol was responsible only for the job it was actually designed to do, outbound failover
started working as expected and the design became noticeably easier to reason about and troubleshoot.
</p>

<h2>Build & Verification Walkthrough</h2>

<p align="center">

Updated topology with DSW2, R2, and the four routed OSPF links: <br/>
<img width="1840" height="1023" alt="image" src="https://github.com/user-attachments/assets/3a4fb3a9-eb96-431b-bbaf-fae264069be3" />
<br /><br />

Successful pings: <br/>
<img width="907" height="410" alt="image" src="https://github.com/user-attachments/assets/ab98fdc5-8917-4ea9-bec8-21cffc23e7d7" />
<img width="927" height="413" alt="image" src="https://github.com/user-attachments/assets/e9676e9c-7aae-48cb-b9f4-ec2ec2e3a822" />
<img width="897" height="413" alt="image" src="https://github.com/user-attachments/assets/8d5a3270-06a9-4b46-9236-cbaac266fa5f" />
<img width="956" height="347" alt="image" src="https://github.com/user-attachments/assets/1593e11e-1ef1-4049-a776-a373f697c612" />
<br /><br />

OSPF neighbor adjacencies fully formed - <code>show ip ospf neighbor</code>: <br/>
<img width="1343" height="207" alt="image" src="https://github.com/user-attachments/assets/8f761d27-dead-41a3-8089-63f8491686b6" />
<img width="1376" height="204" alt="image" src="https://github.com/user-attachments/assets/e004457b-4fb0-4af9-9f51-d4b9cbec50bf" />
<img width="1371" height="299" alt="image" src="https://github.com/user-attachments/assets/e7366e8c-7569-4be1-ad09-1cb9bb6847a4" />
<img width="1342" height="299" alt="image" src="https://github.com/user-attachments/assets/1b608dc2-dc11-4cbd-b4d5-d729daca6567" />
<br /><br />

ISPR1 CONFIG: <br/>
<img width="682" height="857" alt="image" src="https://github.com/user-attachments/assets/bc0040f9-8fb1-4d11-b417-1e4705e30b89" />
<br /><br />

OSPF routing table showing multiple equal-cost paths - <code>show ip route</code>: <br/>
<img width="1259" height="994" alt="image" src="https://github.com/user-attachments/assets/3b7c3c11-12df-4e10-addd-b3eac8a67d6c" />
<img width="1255" height="1349" alt="image" src="https://github.com/user-attachments/assets/e2eb5669-ac43-4e06-afa8-7d16663165ab" />
<br /><br />

HSRP state on DSW1 and DSW2 under normal conditions - <code>show standby brief</code>: <br/>
<img width="1249" height="246" alt="image" src="https://github.com/user-attachments/assets/26c1b8b8-ab1d-4786-880e-a2cf595d79e5" />
<img width="1332" height="237" alt="image" src="https://github.com/user-attachments/assets/2def4f9c-f164-4018-9a6d-c074df25a550" />
<br /><br />

Extended ACL blocking VLAN 10 → VLAN 40 while permitting other traffic - <code>show access-lists</code> with hit counters: <br/>
<img width="1017" height="117" alt="image" src="https://github.com/user-attachments/assets/46fb11e9-8a12-46b9-b958-8eb220981d6e" />

</p>

<h2>Failure Scenarios Tested</h2>

<table>
<tr><th>Scenario</th><th>Expected Behavior</th><th>Mechanism</th></tr>
<tr><td>DSW1 fails</td><td>Hosts keep using the same default gateway IP; DSW2 takes over as active</td><td>HSRP</td></tr>
<tr><td>R1 fails</td><td>Traffic continues to the Internet via R2; default route withdrawn from OSPF</td><td>OSPF</td></tr>
<tr><td>One EtherChannel member link fails</td><td>Port-channel bundle stays up on the remaining member(s)</td><td>LACP</td></tr>
<tr><td>VLAN 10 host attempts to reach VLAN 40</td><td>Traffic is blocked; all other VLAN and Internet traffic unaffected</td><td>Extended ACL</td></tr>
</table>


<br />

<h2>Verification Commands Used</h2>

```cisco
show ip interface brief
show ip route
show ip ospf neighbor
show ip ospf
show standby
show standby brief
show etherchannel summary
show spanning-tree
show access-lists
```
<br />

<h2>Updated Technology Stack (Phase 1 + Phase 2)</h2>

<table>
<tr><th>Technology</th><th>Phase</th><th>Purpose</th></tr>
<tr><td>VLANs / Trunking</td><td>1</td><td>Network segmentation</td></tr>
<tr><td>SVIs</td><td>1</td><td>Inter-VLAN routing</td></tr>
<tr><td>DHCP / DNS</td><td>1</td><td>Address assignment / name resolution</td></tr>
<tr><td>RSTP + PortFast/BPDU Guard</td><td>1</td><td>Layer 2 loop prevention</td></tr>
<tr><td>EtherChannel (LACP)</td><td>1</td><td>Link aggregation and redundancy</td></tr>
<tr><td>Static default routing</td><td>1</td><td>Edge-to-ISP routing (single path)</td></tr>
<tr><td>Redundant distribution/edge devices</td><td>2</td><td>Eliminate single points of failure</td></tr>
<tr><td>HSRP</td><td>2</td><td>First-hop gateway redundancy</td></tr>
<tr><td>Routed P2P links</td><td>2</td><td>Replace shared transit VLAN with direct Layer 3 links</td></tr>
<tr><td>OSPF + ECMP</td><td>2</td><td>Dynamic routing, active/active path forwarding</td></tr>
<tr><td>Loopbacks + OSPF Router ID</td><td>2</td><td>Stable device identity independent of physical interfaces</td></tr>
<tr><td><code>default-information originate</code></td><td>2</td><td>Dynamic propagation of the Internet route</td></tr>
<tr><td>DHCP Relay</td><td>2</td><td>DHCP across a now multi-switch distribution layer</td></tr>
<tr><td>Extended ACL</td><td>2</td><td>Inter-VLAN traffic restriction</td></tr>
</table>
<br />

<h2>What This Adds to the Portfolio💡</h2>
<p> Phase 1 showed I can build and troubleshoot a functioning switched/routed network. Phase 2 shows something a step beyond that: designing for redundancy, and proving it under failure rather than just under ideal conditions. </p>
<p> The core of Phase 2 was adding a second distribution switch and a second edge router so neither layer had a single point of failure. HSRP was the right tool for the first half of that problem — giving the internal network a resilient default gateway, so if one distribution switch went down, the other took over automatically and internal traffic kept flowing without any change on the end devices. </p> 
<p> Extending that same HSRP approach to the routers for outbound traffic was where the design needed to change. Using HSRP between R1 and R2 to protect the path to the Internet added a layer of Layer 2 dependency and complexity that made the desired failover behavior unreliable — it was solving a routing problem with a gateway-redundancy protocol. Replacing that with OSPF across dedicated routed links achieved the actual goal — dynamic, reliable failover to the Internet — with a simpler design and fewer moving parts. Recognizing that HSRP and OSPF solve different problems, and re-architecting around the protocol actually built for the job, was the real lesson of this phase — not just getting a configuration to work, but understanding why one approach was structurally the wrong fit and correcting course. </p>
