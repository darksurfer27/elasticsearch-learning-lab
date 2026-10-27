# 03 — Add Fleet Server on Windows

[← Kibana](02-kibana.md) · [Next: Remote access →](04-remote-access.md)

## Why do I need it?

**Fleet Server** manages Elastic Agents: it tells them which policy to follow and receives agent check-ins. It is **not Elasticsearch**. In this example, it runs on a dedicated Windows computer.

```text
Elastic Agent  <--HTTPS:8220-->  Fleet Server  <--HTTPS:9200-->  Elasticsearch
                   policy + status                          data & state
```

## 1. Understand two different CA certificates

- **Elasticsearch HTTP CA**: allows Fleet Server to trust the Elasticsearch HTTPS endpoint.
- **Fleet Server CA**: allows Elastic Agents to trust Fleet Server's HTTPS endpoint.

Fleet Server additionally needs **its own server certificate and private key**, issued for the actual DNS name/IP used in its URL (matching the certificate SAN).

**Do not upload any real `.key`, `.p12`, service token, or credential to GitHub.** This tutorial doesn't provide real certificates. Generate them for your own setup using the [Elastic TLS guide](https://www.elastic.co/docs/reference/fleet/secure-connections).

## 2. In Kibana, make a Fleet Server policy

1. Open **Management → Fleet → Settings**.
2. Set the **Fleet Server host** to an HTTPS URL with the correct IP or DNS name (training example: `https://192.168.50.30:8220`).
3. Set the default Elasticsearch **output** to a reachable endpoint such as `https://192.168.50.11:9200` and configure its trusted CA/fingerprint correctly.
4. In **Agent policies**, create/select a **Fleet Server Policy** with the **Fleet Server integration**.
5. Copy the **actual policy ID** from the settings/API, not just its human-readable name.

Kibana Dev Tools example (returns policy IDs):

```http
GET kbn:/api/fleet/agent_policies
```

A Fleet Server policy is **not** the same policy you will later give to general Linux monitoring agents.

## 3. Generate a service token and store it privately

In Kibana Dev Tools you can create a token for the `elastic/fleet-server` service account:

```http
POST /_security/service/elastic/fleet-server/credential/token/learning-fleet-server
```

The returned **token value is secret and visible only when generated**. Store it in a protected local text file, not in a GitHub repository, screenshot or copied chat log. If you lose it, create a new token and revoke the unused one.

## 4. Prepare the Windows files

Sample private paths **on your own Windows PC** (not files included in this repository):

```text
C:\FleetLab\certs\fleet-ca.crt
C:\FleetLab\certs\fleet-server.crt
C:\FleetLab\certs\fleet-server.key
C:\FleetLab\certs\es-http-ca.crt
C:\FleetLab\certs\fleet-token.txt
```

Limit access to **Administrators and SYSTEM**, and confirm ACLs using `icacls`. For example:

```cmd
icacls "C:\FleetLab\certs\*"
```

The certificate file and token paths **must exist**, their content must be valid, and the Fleet Server certificate must match the address agents will call.

## 5. Install Elastic Agent as Fleet Server

Download the **Windows Elastic Agent compatible with your stack** from the official Elastic site. Open **CMD as Administrator**, go to the extracted installer folder, and inspect `elastic-agent.exe install --help`.

This example uses the relevant options verified with Elastic Agent 9.x in the original learning lab. **Replace the IPs, paths and policy ID. Do not type `<...>` placeholders literally.** In Windows CMD, `^` must be the final character on a continued line.

```cmd
elastic-agent.exe install ^
  --url=https://192.168.50.30:8220 ^
  --certificate-authorities="C:\FleetLab\certs\fleet-ca.crt" ^
  --fleet-server-es=https://192.168.50.11:9200 ^
  --fleet-server-es-ca="C:\FleetLab\certs\es-http-ca.crt" ^
  --fleet-server-service-token-path="C:\FleetLab\certs\fleet-token.txt" ^
  --fleet-server-policy=YOUR_FLEET_POLICY_ID ^
  --fleet-server-cert="C:\FleetLab\certs\fleet-server.crt" ^
  --fleet-server-cert-key="C:\FleetLab\certs\fleet-server.key" ^
  --fleet-server-host=192.168.50.30 ^
  --fleet-server-port=8220 ^
  --fleet-server-timeout=5m
```

This is a **guided example, not a one-command installer**: certificates, output, policy, token and network access must be prepared first.

## 6. Check it worked

```cmd
"C:\Program Files\Elastic\Agent\elastic-agent.exe" status
```

Look for:

```text
fleet          (HEALTHY) Connected
elastic-agent  (HEALTHY) Running
```

Then test the endpoint with your Fleet Server CA:

```cmd
curl.exe --cacert "C:\FleetLab\certs\fleet-ca.crt" https://192.168.50.30:8220/api/status
```

Expected: `{"name":"fleet-server","status":"HEALTHY"}`. In Kibana → Fleet → Agents, the Windows host should appear as **Healthy**.

## Lessons from troubleshooting

- **Installation times out:** First check certificates, output URL and network, not just the timeout value.
- **`curl (77)` failed to open CA file:** Check certificate path and Windows file permissions (`icacls`).
- **`401 missing authentication credentials` on Elasticsearch:** HTTPS reached the API, but you are not authenticated — often a useful connectivity check.
- **Named pipe `Access is denied`:** Inspect Fleet/Agent logs and Windows process/service state; don't automatically assume it is the certificate issue.
- **Local check-in timeout but `/api/status` healthy:** Investigate Elasticsearch connectivity and the logs; a health endpoint alone doesn't prove check-ins are working.
- **HTTP CA mismatch:** A CA from one Elasticsearch node will not automatically trust another node's different CA.

See [Troubleshooting](05-troubleshooting.md) for a beginner diagnosis flow.
