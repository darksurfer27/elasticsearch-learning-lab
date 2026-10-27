# 04 — Remote access from another network (Tailscale + PuTTY)

[← Fleet Server](03-fleet-server-windows.md) · [Next: Troubleshooting →](05-troubleshooting.md)

## What are we trying to do?

You have a group of computers on a private LAN, and want to reach them from your laptop while away. We will put **Tailscale on just two machines**: your Windows laptop and `es01`, which becomes the **jump host**.

```text
Remote Windows laptop (PuTTY, browser, Tailscale)
       |
       | private Tailscale network + SSH
       v
es01 Linux jump host (Tailscale + SSH)
       | SSH over the local lab network
       +----> es02 (192.168.50.12)
       +----> es03 (192.168.50.13)
       +----> Kibana host (192.168.50.20)
                  | browser tunnel to port 5601
```

**Why not install Tailscale everywhere?** A jump host is a simple beginner method. It requires that es01 can reach the other lab machines on the LAN. It does not make their private IPs directly routable from your Windows laptop.

## 1. Install Tailscale on es01 (Ubuntu)

From an existing trusted session on es01:

```bash
curl -fsSL https://tailscale.com/install.sh -o /tmp/tailscale-install.sh
less /tmp/tailscale-install.sh  # review the script before running it
sudo sh /tmp/tailscale-install.sh
tailscale version
sudo tailscale up
tailscale ip -4
sudo systemctl is-active ssh
```

`tailscale up` displays an **authentication URL**. Open it privately and approve the right machine. `tailscale ip -4` reports your actual Tailscale address: **do not copy a stranger's address**. SSH service should report `active`.

## 2. Connect your Windows laptop

1. Install Tailscale from [the official Windows download](https://tailscale.com/download/windows).
2. Sign into the **same tailnet** and connect.
3. On Windows CMD, run:

```cmd
tailscale status
tailscale ping YOUR_ES01_TAILSCALE_IP
```

**Expected:** es01 appears in the peer list and a `pong` response appears. If you see `no matching peer`, check tailnet selection, device approval and access policies **before** troubleshooting PuTTY.

## 3. SSH to es01 using PuTTY

1. Open PuTTY → **Session**.
2. Host Name: `YOUR_ES01_TAILSCALE_IP` (from Step 1); Port: `22`; connection type: `SSH`.
3. Select **Open**, verify the host key fingerprint on the first connection, and login with your Linux username/password.
4. **Check:** You get a shell prompt such as `learner@es01:~$`.

## 4. SSH from es01 to another lab host

Run these **in the PuTTY window that is already logged into es01**:

```bash
ssh learner2@192.168.50.12  # es02 example
exit                        # return to es01
ssh learner3@192.168.50.13  # es03 example
exit                        # return to es01
ssh kibanauser@192.168.50.20
```

Use the **real account names you created**, not the fictional example users. On your first visit to a host, compare the SSH host-key fingerprint against an independently obtained fingerprint before accepting it.

## 5. Set up `.ppk` key authentication

**Why?** Instead of using your Linux account password to connect from the laptop to es01, PuTTY can use an SSH key.

1. On Windows, open **PuTTYgen** → choose **EdDSA / Ed25519** → **Generate**.
2. Add a strong **key passphrase** → **Save private key** to a local `.ppk` file.
3. Copy the **public key for pasting into OpenSSH authorized_keys file**. Never publish/share the `.ppk` private key.
4. While logged into es01, run:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# paste the PUBLIC key as one complete line; save Ctrl+O Enter, exit Ctrl+X
chmod 600 ~/.ssh/authorized_keys
```

5. Open a **new** PuTTY window (keep your existing connection open). Go to **Connection → SSH → Auth → Credentials** and select your `.ppk` file.
6. Return to **Session** → enter es01's Tailscale IP and your Linux username → **Save** a named session → **Open**.
7. It should prompt for the **key passphrase**, not the Linux account password. Verify public-key login before considering stricter SSH settings.

**Another laptop?** Install Tailscale and PuTTY there, then create a **different SSH key** for that laptop. Register only its public key on es01 so you can revoke access per device.

## 6. Open Kibana in your laptop browser with a tunnel

Kibana's LAN URL might be `http://192.168.50.20:5601`. You cannot simply paste that address into your laptop browser when away from the LAN. Create a **local SSH tunnel** in PuTTY:

1. Open your saved es01 session in PuTTY and choose **Load**.
2. Go to **Connection → SSH → Tunnels**.
3. Set **Source port:** `15601`; **Destination:** `192.168.50.20:5601`; choose **Local** and click **Add**.
4. You should see `L15601 192.168.50.20:5601`.
5. Return to **Session** → **Save** → **Open** → log in to es01 and leave this window connected.
6. On that **same Windows laptop**, visit **http://127.0.0.1:15601** in a browser.
7. Login with a **Kibana web account**, not the Linux SSH credentials.

`127.0.0.1` means **this laptop**. Your Kibana server's actual LAN IP **has not changed**. The SSH tunnel carries the laptop's local port traffic to Kibana through es01. Close PuTTY and the tunnel closes too.

## 7. Small checklist

✅ Windows Tailscale and Linux Tailscale connected · ✅ PuTTY can login to es01 · ✅ es01 can SSH to es02/es03/Kibana · ✅ `.ppk` login works · ✅ browser can open `http://127.0.0.1:15601` **while PuTTY remains connected**.

Never publish your **actual tailnet IP, personal email, SSH private key or Tailscale login/approval link**.
