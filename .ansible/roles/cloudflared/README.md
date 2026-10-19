# cloudflared role

Runs a Cloudflare Tunnel so the server is reachable through Cloudflare's edge
without any inbound ports (works behind CGNAT). Routes `ssh.jgirod.com.ar`
to local SSH on port 2221.

## One-time manual setup (requires your browser + Cloudflare account)

Ansible cannot do these steps: they need an interactive login to YOUR
Cloudflare account. Run them once on the server (as the ansible user):

```bash
# 1. Authenticate this machine to your Cloudflare account (opens a browser link).
cloudflared tunnel login

# 2. Create the named tunnel.
cloudflared tunnel create fortress

# 3. Point the hostname at the tunnel (creates the DNS CNAME in Cloudflare).
cloudflared tunnel route dns fortress ssh.jgirod.com.ar

# 4. Install the credentials where the service expects them.
sudo mkdir -p /etc/cloudflared
sudo cp ~/.cloudflared/*.json /etc/cloudflared/fortress.json
```

Then run the playbook; the role picks up the credentials and starts the service.

## Client side (your collaborator)

Add to their `~/.ssh/config`:

```
Host fortress
    HostName ssh.jgirod.com.ar
    User collaborator
    IdentityFile ~/.ssh/collaborator
    ProxyCommand cloudflared access ssh --hostname %h
```

They need `cloudflared` installed locally. Then: `ssh fortress`.
