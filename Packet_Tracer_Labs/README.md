# Cisco Packet Tracer Networking Labs

A collection of five Cisco Packet Tracer practice labs covering DHCP, Telnet remote management, NTP time synchronization, printer sharing, and TFTP file transfer.

## Repository contents

| Lab | File | What it demonstrates |
|---|---|---|
| 01 — DHCP | `01-dhcp/DHCP_LAB.pkt` | Assigning IP configuration to clients automatically using DHCP |
| 02 — Telnet | `02-telnet/TELNET_LAB.pkt` | Remotely accessing a Cisco device through a VTY line |
| 03 — NTP | `03-ntp/NTP_LAB.pkt` | Synchronizing network-device time with an NTP time source |
| 04 — Printer Sharing | `04-printer-sharing/PRINTER-SHARING_LAB.pkt` | Connecting clients to a network printer and checking reachability |
| 05 — TFTP | `05-tftp/TFTP_LAB.pkt` | Transferring or backing up a device configuration to a TFTP server |

## Requirements

- Cisco Packet Tracer (a version that can open the included `.pkt` files).
- Basic familiarity with router/switch CLI and end-device IP configuration.
- Open each `.pkt` file in Packet Tracer and inspect the labels, links, device interfaces, and IP addressing before entering commands.

## Important addressing note

The `.pkt` files are native Packet Tracer project files. This README intentionally does **not** claim specific device names, interface numbers, IP addresses, or successful simulation results that have not been independently verified from the open projects. IP addresses and interface names shown in the command examples below are examples only. Replace them with the addresses and interfaces used in the corresponding topology.

## How to use the labs

1. Download or clone this repository.
2. Open the relevant `.pkt` file in Cisco Packet Tracer.
3. Review the topology and note each device's IP address, subnet mask, gateway, and connected interface.
4. Complete or check the configuration described in that lab's section below.
5. Use the listed verification commands and end-device tests.
6. Save a copy of the `.pkt` file after making changes, so the original exercise remains available.

---

## Lab 01 — DHCP

**File:** `01-dhcp/DHCP_LAB.pkt`

### Objective
Configure or verify Dynamic Host Configuration Protocol (DHCP), which leases IP settings to clients automatically instead of requiring every client to be configured manually.

### Configuration outline

First identify whether the topology uses a **router as the DHCP server** or a **Server-PT device**. Follow the appropriate method and use the subnet shown in the Packet Tracer topology.

#### Option A: Cisco router provides DHCP

Example only, assuming a LAN of `192.168.1.0/24` and router gateway `192.168.1.1`:

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

Change `gigabitEthernet0/0`, the subnet, gateway, and excluded range to match the file. Do not configure the same LAN gateway address on both the router and a server.

#### Option B: Server-PT provides DHCP

1. Open the server and set a **static** IP address, subnet mask, and gateway that match the topology.
2. Open **Services > DHCP**, turn the DHCP service **On**, and configure the pool's start IP, subnet mask, default gateway, DNS server, and maximum number of users.
3. On each client, open **Desktop > IP Configuration** and select **DHCP**.

### Verification

- Client devices receive an IP address, subnet mask, gateway, and (if configured) DNS server.
- The assigned address belongs to the intended subnet and is not a duplicate of a statically configured address.
- Test communication with `ping <default-gateway-IP>`.
- On a router DHCP server, check `show ip dhcp binding` and `show ip dhcp pool`.

---

## Lab 02 — Telnet Remote Access

**File:** `02-telnet/TELNET_LAB.pkt`

### Objective
Enable remote CLI access to a Cisco router or switch through Telnet and verify that privileged EXEC access is protected by the enable password.

### Credentials for this lab

- **VTY/ Telnet line password:** `cisco`
- **Enable password:** `class`
- **Username:** No username was specified for this lab; the example uses line-password authentication (`login`).

These are lab credentials, not suitable for a production network. Telnet transmits credentials in clear text; use SSH for real systems.

### Example configuration

Apply this on the target router or switch. Replace the VTY range and management IP as needed for the device in the file.

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

If the device supports a different VTY range, configure the applicable lines. The target device must also have a reachable IP address, and the client must be able to reach that address.

### Verification

1. From a PC, open **Desktop > Command Prompt**.
2. Run `ping <device-management-IP>` first.
3. Run `telnet <device-management-IP>`.
4. At the Telnet password prompt, enter `cisco`.
5. At the device prompt, enter `enable`; when asked for the enable password, enter `class`.
6. Verify that the prompt changes to privileged EXEC mode (for example, `Router#` or `Switch#`).

If Telnet fails, check the device IP, interface status, VTY password and `login` setting, permitted transport, and connectivity between the PC and the device.

---

## Lab 03 — NTP Time Synchronization

**File:** `03-ntp/NTP_LAB.pkt`

### Objective
Use Network Time Protocol (NTP) to keep network-device clocks synchronized with a time source.

### Configuration outline

First identify the intended NTP server in the topology. If it is a **Server-PT device**, set its IP configuration and enable its NTP service under **Services > NTP**. If a Cisco router is intended to act as the time source, the lab may instead use the router's NTP master function.

#### Cisco device acting as an NTP client

```text
enable
configure terminal
ntp server <NTP-SERVER-IP>
end
show clock
show ntp associations
show ntp status
```

Replace `<NTP-SERVER-IP>` with the time server's actual IP address.

#### Optional: Cisco router acting as the NTP source

Only use this if the topology requires the router to provide time:

```text
enable
configure terminal
ntp master
end
```

Configure other devices to use that router's reachable IP address with `ntp server <ROUTER-IP>`.

### Verification

- Confirm the client can reach the configured time source.
- Check `show ntp associations` and `show ntp status` on the Cisco device.
- Compare `show clock` before and after synchronization. NTP may take time to establish synchronization, so an immediate unsynchronized status does not always indicate a configuration error.

---

## Lab 04 — Printer Sharing

**File:** `04-printer-sharing/PRINTER-SHARING_LAB.pkt`

### Objective
Connect multiple end devices to a network printer and verify that the printer is reachable on the LAN.

### Configuration outline

1. Open the topology and identify the printer, clients, switch, and any router involved.
2. Assign the printer an IP address and subnet mask appropriate for the network. Use a static address or a DHCP reservation/lease according to the exercise design.
3. Make sure each client has a unique IP address in the correct subnet and the correct subnet mask.
4. If clients and the printer are on different subnets, configure the appropriate default gateways and routing. A gateway is normally unnecessary for communication confined to the same subnet.
5. In Packet Tracer, inspect the printer's available configuration options and any printer service or printing application supported by that device model/version.

### Verification

- Check that the printer and clients are connected to the correct network and have valid IP settings.
- From each client, run `ping <printer-IP>` if ICMP is supported by the device.
- Check the switch links and interface status if connectivity fails.
- If ping succeeds but printing does not, verify the printer/application setup supported by the Packet Tracer device; ping alone proves reachability, not successful printing.

---

## Lab 05 — TFTP File Transfer / Configuration Backup

**File:** `05-tftp/TFTP_LAB.pkt`

### Objective
Use Trivial File Transfer Protocol (TFTP) to transfer files between a Cisco network device and a TFTP server. A common use is backing up a router or switch configuration.

### Server setup

1. Give the Server-PT device a static IP address and subnet mask appropriate for the topology.
2. Set its default gateway if the transfer crosses a subnet.
3. Open **Services > TFTP** and turn the TFTP service **On**.
4. Confirm that the Cisco device can reach the server using `ping <TFTP-SERVER-IP>`.

### Back up a Cisco device configuration

On the router or switch, use the server IP shown in your topology:

```text
enable
copy running-config tftp:
```

When prompted, enter the TFTP server's IP address and a meaningful destination filename, for example `router-running.cfg`.

To back up the saved startup configuration instead, use:

```text
copy startup-config tftp:
```

### Verification and restore caution

- Check the Server-PT TFTP service/file list, if supported by the Packet Tracer version, to confirm the file arrived.
- Confirm the filename and server address used by the transfer.
- If practising a restore, back up the current configuration first and make sure the file is the intended one. A restore can change device settings and may interrupt connectivity.

---

## Common troubleshooting checklist

- **No connectivity:** verify cables, link lights, interface status (`show ip interface brief` on Cisco devices), IP addresses, subnet masks, and gateways.
- **DHCP client gets no address:** confirm the DHCP service/pool is enabled, the pool matches the client LAN, the router interface is up, and DHCP requests can reach the server.
- **Telnet fails:** verify IP reachability, VTY password plus `login`, and the permitted transport protocol.
- **NTP is not synchronized:** verify the server IP, routing/reachability, enabled NTP service, and allow time for synchronization.
- **Printer is unreachable:** check the printer's IP configuration, the client subnet, switch connections, and routing if different subnets are used.
- **TFTP transfer fails:** verify the server's IP, TFTP service status, IP connectivity, filename prompts, and that the selected server is the TFTP server.

## Scope and validation

This repository contains Packet Tracer project files and a lab guide. The command snippets and example IP addresses are reference configurations, not proof that each project has those exact settings. Open each `.pkt` file in Cisco Packet Tracer, compare the instructions against its actual topology, run the verification steps, and save any validated changes before presenting the work as completed.
