# Cisco Packet Tracer Networking Labs

A collection of eight Cisco Packet Tracer practice projects covering IP addressing, routing, network services, and basic LAN connectivity. The projects are intended for hands-on learning: open the `.pkt` file in Cisco Packet Tracer, inspect its topology and instructions, complete or validate the configuration, and test the result.

## Repository contents

| # | Lab | Project file | Main learning goal |
|---:|---|---|---|
| 01 | DHCP | `01-dhcp/DHCP_LAB.pkt` | Automatically assign IP configuration to clients |
| 02 | Telnet | `02-telnet/TELNET_LAB.pkt` | Remotely access a Cisco device through VTY lines |
| 03 | NTP | `03-ntp/NTP_LAB.pkt` | Synchronize network-device clocks with a time source |
| 04 | Printer sharing | `04-printer-sharing/PRINTER-SHARING_LAB.pkt` | Connect clients to a network printer and test reachability |
| 05 | TFTP | `05-tftp/TFTP_LAB.pkt` | Transfer or back up network-device files using TFTP |
| 06 | Static routing | `06-static-routing/STATIC ROUTING 2.pkt` | Configure routes manually so networks behind different routers can communicate |
| 07 | Subnetting — `18` | `07-subnetting-18/lab_subnetting-18.pkt` | Practice subnet calculations; the filename may indicate a `/18` prefix exercise |
| 08 | Subnetting — `26` | `08-subnetting-26/lab_subnetting-26.pkt` | Practice subnet calculations; the filename may indicate a `/26` prefix exercise |

> **Topology and addressing note:** `.pkt` files are native Packet Tracer project files. Their internal topology, notes, device interfaces, and address tables could not be independently extracted as readable text in this environment. Therefore, this guide does not pretend to know the exact routers, interfaces, IP addresses, or completed simulation results in the uploaded projects. Check the labels and instructions inside each file before applying commands. The `/18` and `/26` descriptions above are based on the filenames; confirm that these are the intended prefix lengths in Packet Tracer.

## Requirements

- Cisco Packet Tracer version compatible with the included `.pkt` files.
- Basic understanding of IPv4 addresses, subnet masks, default gateways, and the Cisco IOS CLI.
- For CLI labs, use the device's **CLI** tab. For end devices, use **Desktop > IP Configuration** and **Desktop > Command Prompt** where available.

## How to use a lab

1. Open the matching `.pkt` file in Packet Tracer.
2. Read all on-canvas instructions and inspect device names, links, port labels, and IP-address notes.
3. Record the actual IP addresses, subnet masks, gateways, and interface names shown in the project.
4. Apply only the commands relevant to that topology. Commands and address examples in this README are reference patterns, not a substitute for the addressing plan in the file.
5. Verify connectivity and configuration using the checks listed under that lab.
6. Save your completed work under a new filename if you want to preserve the original exercise.

---

## Lab 01 — DHCP

**Project file:** `01-dhcp/DHCP_LAB.pkt`

### Objective

Dynamic Host Configuration Protocol (DHCP) leases IP configuration to clients, avoiding the need to configure every PC manually.

### What to inspect and configure

First identify whether a **Cisco router** or a **Server-PT** device is the DHCP server. Use only the matching option below and substitute the subnet and interface from the topology.

#### Option A: Cisco router provides DHCP

The following is a syntax example only for a hypothetical `192.168.1.0/24` LAN with gateway `192.168.1.1`:

```text
enable
configure terminal
interface gigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit
ip dhcp excluded-address 192.168.1.1 192.168.1.20
ip dhcp pool LAN
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
exit
end
show ip dhcp pool
show ip dhcp binding
```

Change the sample interface, network, gateway, and excluded range to match the project. Do not assign the same IP address to the router and a server.

#### Option B: Server-PT provides DHCP

1. Give the server a static IP address, mask, and gateway matching the topology.
2. Open **Services > DHCP**, turn the service **On**, and enter the pool start address, mask, gateway, DNS server, and maximum number of users.
3. On each client, open **Desktop > IP Configuration** and choose **DHCP**.

### Verify

- Clients receive an address and mask from the intended subnet.
- The default gateway and DNS settings are correct for the exercise.
- Ping the gateway from a client.
- On a router DHCP server, use `show ip dhcp binding` and `show ip dhcp pool`.

---

## Lab 02 — Telnet remote access

**Project file:** `02-telnet/TELNET_LAB.pkt`

### Objective

Connect to a Cisco device's command line remotely over Telnet and enter privileged EXEC mode.

### Credentials specified for this lab

- **VTY/Telnet line password:** `cisco`
- **Enable password:** `class`
- **Username:** none specified; the documented method is line-password authentication.

These are lab credentials only. Telnet sends credentials in clear text and should not be used for sensitive production administration; use SSH in real networks.

### Example VTY configuration

Use this as a reference and adjust the line range if the target device shows a different range:

```text
enable
configure terminal
enable password class
line vty 0 4
 password cisco
 login
 transport input telnet
exit
end
write memory
```

The target device also needs a reachable management IP address and an active interface. Do not add `login local` unless you intentionally configure username-based authentication.

### Verify

1. From a PC, open **Desktop > Command Prompt** and ping the target management IP.
2. Run `telnet <device-management-IP>`.
3. At the Telnet password prompt, enter `cisco`.
4. Enter `enable`; at the enable-password prompt, enter `class`.
5. Confirm that the prompt changes to privileged EXEC mode, typically ending with `#`.

If it fails, check addressing, reachability, VTY password and `login`, and whether Telnet is permitted on the VTY lines.

---

## Lab 03 — NTP time synchronization

**Project file:** `03-ntp/NTP_LAB.pkt`

### Objective

Use Network Time Protocol (NTP) to synchronize a router or switch clock with a time source.

### What to inspect and configure

Identify the time source marked in the topology. It may be a Server-PT device or a Cisco device configured as an NTP master.

On a Cisco device acting as an NTP client, configure the reachable server IP shown in the project:

```text
enable
configure terminal
ntp server <NTP-SERVER-IP>
end
show clock
show ntp associations
show ntp status
```

If the topology explicitly calls for a Cisco router to serve time, the router may use this configuration:

```text
enable
configure terminal
ntp master
end
```

On other devices, configure `ntp server <ROUTER-IP>` using that router's reachable address. If using Server-PT, check **Services > NTP** and enable/configure the service as directed by the lab.

### Verify

- The client can reach the intended time source.
- `show ntp associations` and `show ntp status` indicate whether synchronization has been established.
- Check `show clock` again after giving NTP time to synchronize.

---

## Lab 04 — Printer sharing

**Project file:** `04-printer-sharing/PRINTER-SHARING_LAB.pkt`

### Objective

Connect a network printer and client devices to the LAN, then verify the printer's reachability from the clients.

### What to inspect and configure

1. Identify the printer, PCs, switch, and any router shown in the diagram.
2. Check the printer's IP address and subnet mask. Use the addressing provided in the project; if the exercise requires assigning an address, ensure it does not conflict with another device.
3. Check that each PC has a unique IP address in the correct subnet.
4. If the printer and clients are in different subnets, confirm that their gateways and routing are configured. A gateway normally is not needed for devices communicating within the same subnet.
5. Inspect the printer settings and any printing application available for the selected Packet Tracer device/version.

### Verify

- Link indicators and IP settings are correct.
- From each client, try `ping <printer-IP>` if the Packet Tracer printer model responds to ICMP.
- If ping succeeds but the print test fails, check the supported printer/application configuration. Successful ping demonstrates network reachability, not necessarily successful printing.

---

## Lab 05 — TFTP file transfer / configuration backup

**Project file:** `05-tftp/TFTP_LAB.pkt`

### Objective

Use Trivial File Transfer Protocol (TFTP) to transfer a file between a Cisco device and a TFTP server. A common practice task is backing up a running or startup configuration.

### Server setup

1. Configure the Server-PT device with a static IP address and subnet mask from the topology.
2. Set the gateway if communication crosses a subnet.
3. Open **Services > TFTP** and turn the service **On**.
4. From the router or switch, ping the server to confirm reachability.

### Back up a configuration

On the Cisco router or switch, run:

```text
enable
copy running-config tftp:
```

At the prompts, enter the actual TFTP server IP and a destination filename, such as `router-running.cfg`. To copy the saved startup configuration instead, use:

```text
copy startup-config tftp:
```

### Verify and use caution when restoring

Check the Server-PT TFTP service/file list, if provided by the Packet Tracer version, and confirm the filename. Before practising a restore, back up the current configuration and make sure the selected file is the intended one; restoring a configuration can change settings or disrupt connectivity.

---

## Lab 06 — Static routing

**Project file:** `06-static-routing/STATIC ROUTING 2.pkt`

### Objective

Static routing uses routes entered manually by an administrator. A router needs a route to each remote destination network, either through a next-hop IP address or an outgoing interface. A working end-to-end path generally requires a forward route and a return route.

### Steps

1. Open the project and count the routers and LANs. Record every router interface IP/mask and the networks attached to it.
2. Check that directly connected interfaces are addressed and enabled.
3. For each router, identify remote destination networks not already shown as directly connected.
4. Add the appropriate static route using the actual destination network, mask, and next hop from the topology.
5. Add return routes on the other routers as needed. A route in only one direction may allow a packet to travel out but not receive a reply.

### Cisco IOS route syntax

```text
configure terminal
ip route <DESTINATION-NETWORK> <SUBNET-MASK> <NEXT-HOP-IP>
end
show ip route
show ip route static
show ip interface brief
```

The angle-bracket values are placeholders, not commands to enter literally. If the activity directs you to use an outgoing interface, use the interface-based `ip route` form supported by that device and topology. Do not copy an example route until you have checked the project addressing.

### Verify

- `show ip interface brief` shows the required interfaces as up/up.
- `show ip route` contains the expected connected networks and static routes (static routes are commonly marked `S`).
- Ping from a host in one LAN to a host in a remote LAN.
- If the ping fails, check the local gateway, each router's route to the destination, the return path, masks, and interface status.
- Use `traceroute <destination-IP>` from a Cisco CLI, or the corresponding end-device command if supported, to help locate where traffic stops.

### Common mistakes

- Using a next-hop IP that is not reachable through a directly connected network.
- Entering the destination host IP instead of the destination network and its mask.
- Forgetting the return route on a remote router.
- Using the wrong subnet mask or leaving an interface shut down.

---

## Lab 07 — Subnetting practice (`lab_subnetting-18.pkt`)

**Project file:** `07-subnetting-18/lab_subnetting-18.pkt`

### Objective

Practise interpreting a prefix length and calculating the subnet mask, network address, broadcast address, usable host range, and number of usable hosts. The filename suggests a `/18` prefix exercise; confirm the actual target prefix and base IP address from the project labels.

### `/18` reference calculations

| Property | Value for a `/18` IPv4 prefix |
|---|---|
| Subnet mask | `255.255.192.0` |
| Network bits / host bits | 18 / 14 |
| Total addresses per subnet | `2^14 = 16,384` |
| Usable host addresses (traditional IPv4 subnet) | `16,382` |
| Wildcard mask | `0.0.63.255` |
| Block size in the third octet | `256 - 192 = 64` |

**Illustrative example only:** for `172.16.0.0/18`, the subnet spans `172.16.0.0` through `172.16.63.255`. The network address is `172.16.0.0`, the broadcast address is `172.16.63.255`, and the traditional usable host range is `172.16.0.1` through `172.16.63.254`. Use the actual base address in the Packet Tracer exercise for its answer.

### Subnetting workflow

1. Read the IP address and prefix length provided by the activity.
2. Convert the prefix to a subnet mask. For `/18`, 18 bits are network bits and 14 are host bits.
3. Find the interesting octet (the first mask octet that is not `255` or `0`) and calculate the block size.
4. Find the subnet block containing the given IP; the block start is the network address.
5. The address immediately before the next block is the broadcast address. The first and last addresses between network and broadcast are the usable range in a traditional subnet.
6. Check the calculation against any labels or address table included in the `.pkt` project.

### Verify in Packet Tracer

If the file includes configured devices, check that each IP address is paired with the correct mask. Devices intended to share a subnet should have compatible network addresses. If the activity is a calculation-only exercise, record the calculated mask, network, broadcast, usable range, and host count in your notes instead of inventing device settings.

---

## Lab 08 — Subnetting practice (`lab_subnetting-26.pkt`)

**Project file:** `08-subnetting-26/lab_subnetting-26.pkt`

### Objective

Practise calculating subnet boundaries and host ranges for a smaller IPv4 subnet. The filename suggests a `/26` prefix exercise; verify the actual prefix and base address in Packet Tracer before treating this reference as the project's answer.

### `/26` reference calculations

| Property | Value for a `/26` IPv4 prefix |
|---|---|
| Subnet mask | `255.255.255.192` |
| Network bits / host bits | 26 / 6 |
| Total addresses per subnet | `2^6 = 64` |
| Usable host addresses (traditional IPv4 subnet) | `62` |
| Wildcard mask | `0.0.0.63` |
| Block size in the fourth octet | `256 - 192 = 64` |

**Illustrative example only:** within `192.168.10.0/24`, the first `/26` subnet is `192.168.10.0/26`: network `192.168.10.0`, usable hosts `192.168.10.1`–`192.168.10.62`, and broadcast `192.168.10.63`. The next `/26` subnet starts at `192.168.10.64`, has usable hosts `.65`–`.126`, and broadcast `.127`. These are teaching examples, not claims about the exact IPs in the uploaded file.

### Subnetting workflow

1. Convert `/26` to `255.255.255.192`.
2. Compute the block size: `256 - 192 = 64` addresses per subnet.
3. Within the relevant `/24`, subnet network addresses therefore start at `.0`, `.64`, `.128`, and `.192`.
4. For the block containing the given address, identify the network address, next block, broadcast (one less than the next block), and usable range.
5. Confirm the calculated subnet against the activity's actual IP and prefix.

### Verify in Packet Tracer

Check the IP address and mask on each configured device. Devices expected to communicate directly must be in the same subnet; devices in different subnets need a router and suitable routing. If the project only asks for calculations, document the results shown by the exercise rather than changing the topology unnecessarily.

---

## Useful Cisco verification commands

Use only the commands supported by the device model in the project:

```text
show ip interface brief
show running-config
show ip route
show ip route static
ping <DESTINATION-IP>
traceroute <DESTINATION-IP>
```

Not every Packet Tracer end device supports the same CLI commands. On PCs, use **Desktop > Command Prompt** and the commands available there.

## Troubleshooting checklist

- **No link/connectivity:** check cables, link indicators, interface status, addressing, and masks.
- **Wrong subnet or no hosts:** recalculate the mask and network boundary; make sure the IP is not the network or broadcast address.
- **DHCP failure:** check that the service/pool is on, the pool matches the LAN, and the server or router can receive DHCP requests.
- **Telnet failure:** check management-IP reachability, VTY password plus `login`, and whether Telnet is permitted.
- **NTP not synchronized:** verify server address, service status, routing, and allow time for synchronization.
- **Printer unreachable:** verify printer/client IP settings and routing if they are in different networks.
- **TFTP failure:** check server address, TFTP service status, reachability, and the entered filename.
- **Static route not working:** confirm the next hop is reachable and that all routers have the required forward and return routes.

## Validation note

The `.pkt` files should be opened and tested in Cisco Packet Tracer before claiming that a lab is complete or that a command sequence matches its exact topology. This README provides objectives, reference syntax, calculation examples, and checks; it does not claim that unverified simulations have passed.
