# 06 — Next exercise: enroll Elastic Agents on Linux hosts

[← Troubleshooting](05-troubleshooting.md) · [Next: Publish →](07-publish-to-github.md)

> **Status: NOT YET COMPLETED in the documented original lab.** This is a future exercise, not a claim of completed experience.

## What is the difference between Fleet Server and an ordinary agent?

- **Fleet Server**: coordination and management for agents.
- **Elastic Agent**: collects logs, metrics, and integration-specific information and reports to Fleet Server.

## Suggested beginner sequence

1. In **Kibana → Management → Fleet → Agent policies**, create `Linux Servers Policy`. Do **not** enroll generic Linux hosts into the Fleet Server policy.
2. In **Fleet → Agents → Add agent**, select the new Linux policy.
3. Make sure the Linux host can reach **Fleet Server on HTTPS 8220** and **Elasticsearch on HTTPS 9200** (the chosen output endpoint).
4. Download/install the Agent version compatible with the Elastic Stack using **Kibana's current OS-specific instructions**.
5. For a privately issued Fleet Server certificate, provide the **trusted CA** using `--certificate-authorities=/path/to/fleet-ca.crt` as documented by Elastic.
6. Enroll **one Linux node first** and verify it appears in Fleet as `Healthy` before proceeding to the others.
7. Add the appropriate **System integration** and check which datasets you actually collect. Review resource overhead and access permissions on Elasticsearch nodes.

Why no enrollment command here? The enrollment token is **secret**, the correct package differs by OS/architecture, and the user interface generates current commands based on the policy and version. Copy the command from **your own Kibana**, never from a public README containing someone's token.

See the [official Elastic Agent guide](https://www.elastic.co/docs/reference/fleet/install-fleet-managed-elastic-agent).
