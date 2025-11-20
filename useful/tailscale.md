<!-- ...existing content... -->

## Instructions

To bring up Tailscale in an unprivileged container, enable access to the /dev/tun device in the LXC config. For example, on Proxmox 7.0 hosting an unprivileged LXC with ID 112, add the following lines to /etc/pve/lxc/112.conf:

```ini
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

If the LXC is already running, shut it down and start it again for the change to take effect. Once /dev/tun is available inside the container, install the Tailscale Linux package within the LXC and authenticate as usual.

## Tailscale on a Proxmox host

### Access to the Proxmox Web UI via Tailscale

The following script generates a certificate using Tailscale and installs it in Proxmox. Can be run this manually or add it as a cron job to keep the certificate up to date.

```bash
#!/bin/bash
NAME="$(tailscale status --json | jq '.Self.DNSName | .[:-1]' -r)"
tailscale cert "${NAME}"
pvenode cert set "${NAME}.crt" "${NAME}.key" --force --restart
```

### Using Tailscale Serve to proxy the Web UI

Serve can provide a valid certificate automatically and let you omit the port number from the URL by proxying on the default HTTPS port (443):

```bash
sudo tailscale serve --bg https+insecure://localhost:8006
```

With the --bg flag  Serve runs as a background process.
