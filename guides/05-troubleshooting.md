# 05 — Troubleshooting: symptoms, meaning, next check

[← Remote access](04-remote-access.md) · [Next: Linux Agents →](06-next-steps-linux-agents.md)

This table comes from **types of problems encountered during the learning lab**. Errors can have multiple causes; test each hypothesis rather than changing several unrelated settings at once.

| Symptom | What it may mean | Safe first test |
| --- | --- | --- |
| `curl: (77) ... failed to open CA file` on Windows | CA file missing, wrong path or unreadable ACL | `dir "C:\...\ca.crt"` and `icacls "C:\...\ca.crt"` |
| HTTP `401 missing authentication credentials` | HTTPS endpoint reached; login not supplied | Retry with an authorized user, without placing passwords in command history |
| `x509: certificate signed by unknown authority` | Wrong/missing CA trust | Confirm certificate issuer and the CA file used by the client |
| Fleet Server installation times out | Fleet Server not ready or output blocked | Check Fleet logs, CA paths, Elasticsearch reachability and service token |
| `named pipe ... Access is denied` | Windows local process/API permission or conflict | Examine installed service identity and relevant Agent/Fleet logs; don't assume TLS root cause |
| `checkin ... timeout awaiting headers` | Check-in handler slow or dependent backend unreachable | Test `/api/status` *and* connectivity from Fleet Server to Elasticsearch |
| `dial tcp ... unreachable host` | Network path temporarily unavailable | `ping`/`curl`/firewall tests from the machine producing the error |
| `PuTTY Network error: Connection timed out` | SSH endpoint unreachable over network | `tailscale status`, `tailscale ping`, verify destination IP and SSH listener |
| `tailscale ping ... no matching peer` | Device not in client's reachable tailnet peer list | Check signed-in tailnet, device authorization, and access policy |
| Elasticsearch only shows 1 or 2 nodes | Discovery, transport TLS, port 9300 or bootstrap issue | `getent hosts ...`, Elasticsearch log, `_cat/nodes` |

## The useful commands and why they matter

**Linux:**

```bash
sudo systemctl status elasticsearch --no-pager
sudo journalctl -u elasticsearch -n 60 --no-pager
getent hosts es01 es02 es03
```

**Windows Fleet Server:**

```cmd
sc query "Elastic Agent"
"C:\Program Files\Elastic\Agent\elastic-agent.exe" status
tasklist /FI "IMAGENAME eq fleet-server.exe"
netstat -ano | findstr ":8220"
netstat -ano | findstr ":8221"
```

**SSH and tunnel:**

```cmd
tailscale status
tailscale ping YOUR_ES01_TAILSCALE_IP
```

## Important interpretation

A **200 HEALTHY** response from `/api/status` only proves that endpoint responds. Fleet Agent check-ins can **still** be failing if Elasticsearch is unavailable. In a real debug session the status changed over time, so always check *current* logs and timestamps before concluding an older error is ongoing.

**Do not publish raw logs**: they can expose hostnames, account IDs, user details, IPs, token names or other infrastructure information. Share short redacted excerpts instead.
