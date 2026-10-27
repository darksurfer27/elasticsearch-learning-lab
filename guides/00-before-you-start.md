# 00 — Before you start

[← Home](../README.md) · [Next: Elasticsearch →](01-elasticsearch-cluster.md)

## What is the goal?

We want a small **private** learning network with three Linux servers for Elasticsearch, one Kibana server, and an optional Windows Fleet Server. The separate remote-access guide shows how to reach this lab from a laptop.

## 1. Gather what you need

- Three Linux computers or VMs: this sample uses **Ubuntu for es01/es03** and **Rocky Linux for es02**.
- Another Linux computer or VM for **Kibana**, plus a Windows computer for **Fleet Server** if following that part.
- An administrator account (`sudo` on Linux, Administrator on Windows).
- A local network where the machines can reach one another, and sufficient RAM/disk for the version you install.
- A plan for SSH access and a secure way to store passwords and certificate material.

Check the **current requirements and supported OS** in official Elastic documentation; hardware and kernel requirements depend on version. For a low-memory learning lab, watch out for virtual-machine RAM and disk usage.

## 2. Plan your own addresses

These are **made-up training values** to show how the hosts relate:

| Role | Example host | Example LAN IP | What runs there |
| --- | --- | --- | --- |
| Elasticsearch node 1 | `es01` | `192.168.50.11` | Ubuntu + Elasticsearch |
| Elasticsearch node 2 | `es02` | `192.168.50.12` | Rocky Linux + Elasticsearch |
| Elasticsearch node 3 | `es03` | `192.168.50.13` | Ubuntu + Elasticsearch |
| Kibana | `kibana01` | `192.168.50.20` | Kibana web UI |
| Fleet Server | `fleetwin01` | `192.168.50.30` | Windows + Elastic Agent/Fleet Server |

**Replace these with real addresses from your private subnet.** If your router doesn't use `192.168.50.x`, these commands will not work unchanged. Use DHCP reservations or static IPs configured correctly for your network.

## 3. Verify on each Linux server

```bash
hostnamectl            # What is this computer called?
hostname -I            # What are its network IP addresses?
free -h                # How much RAM is available?
df -h                  # Is there enough disk space?
```

If you use `/etc/hosts` instead of local DNS, entries might look like:

```text
192.168.50.11 es01
192.168.50.12 es02
192.168.50.13 es03
```

**Check:** `getent hosts es01 es02 es03` prints the addresses you intended. Repeat on all three nodes.

## 4. Know which ports matter

| Port | From → to | Why |
| --- | --- | --- |
| `22` | Your trusted admin machine → Linux | SSH administration |
| `9200` | Kibana / Fleet Server / allowed clients → Elasticsearch | HTTPS API |
| `9300` | Elasticsearch nodes ↔ each other | Node-to-node traffic |
| `5601` | Your trusted browser / SSH tunnel → Kibana | Kibana UI |
| `8220` | Elastic Agents → Fleet Server | Agent management over HTTPS |

Only allow these flows from machines that need them. **Do not port-forward them from your home router to the public internet.**

## Before going on

✅ Every node has a unique hostname; ✅ all nodes resolve the others; ✅ network is reachable; ✅ you have a backup/recovery plan for your own data and credentials.
