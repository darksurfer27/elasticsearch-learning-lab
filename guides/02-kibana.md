# 02 — Add Kibana: the web dashboard

[← Elasticsearch](01-elasticsearch-cluster.md) · [Next: Fleet Server →](03-fleet-server-windows.md)

## What does Kibana do?

Kibana is the **web interface** for Elasticsearch: search data, build dashboards, inspect cluster status, and use Fleet. Kibana runs as its own service, even if Elasticsearch runs on different computers.

## Step 1 — Install the matching Kibana release

On the separate Linux Kibana host, follow the **official Kibana installation guide for your OS** and use a version compatible with Elasticsearch. Do not mix untested prerelease and stable builds. Example for supported RPM repository systems after configuring the Elastic 9.x repo:

```bash
sudo dnf install kibana
```

For Ubuntu, follow the official Kibana APT instructions. Details differ by operating system and installed versions.

## Step 2 — Configure the connection securely

Kibana uses `/etc/kibana/kibana.yml` on package installations. Key ideas:

```yaml
server.host: "192.168.50.20"
server.port: 5601
elasticsearch.hosts: ["https://192.168.50.11:9200"]
elasticsearch.ssl.certificateAuthorities: ["/etc/kibana/certs/es-http-ca.crt"]
# Use a Kibana service-account token or official enrollment mechanism.
# Do NOT paste an actual token into this example file or GitHub!
```

See [`kibana.yml.example`](../examples/kibana.yml.example). You can use a **Kibana service-account token** created through Elastic's official workflow, stored in your actual private configuration. The Fleet Server token is a **different credential**; don't reuse it for Kibana.

If you add three Elasticsearch HTTPS URLs, ensure Kibana trusts the CA certificates for **all three** nodes. If trust is incomplete, start with one reachable URL while you correct certificate trust.

## Step 3 — Start and check

```bash
sudo systemctl enable --now kibana
sudo systemctl status kibana --no-pager
sudo journalctl -u kibana -n 40 --no-pager
```

**Expected result:** open `http://192.168.50.20:5601` **from within your trusted LAN** and see the Kibana login page. Login using an Elasticsearch/Kibana user; this is **not** a Linux SSH username.

> HTTP from a browser is used in this example for a **trusted private learning network only**. Do not expose this HTTP port publicly. For access from outside your LAN, use the [SSH tunnel guide](04-remote-access.md) or deploy HTTPS correctly.
