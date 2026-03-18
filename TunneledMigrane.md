# Fixing HTTPS in a Homelab: Wildcard Certs, Cloudflare Tunnels, and Why It Was Never NPM's Fault

If you run Nginx Proxy Manager behind a Cloudflare tunnel and keep getting SSL errors, redirect loops, 502s, or 1033 tunnel errors — this is probably your problem too. Some of it, anyway. This article covers two separate debugging sessions on the same stack, each one surfacing a different failure mode. They're documented together because each fix created the conditions for the next problem.

## The Stack

- **Nginx Proxy Manager (NPM)** — internal reverse proxy for all services
- **Pi-hole** — local DNS, all subdomains on a custom TLD resolve to NPM internally
- **Cloudflare Tunnel (cloudflared)** — public access without port forwarding, running in a dedicated LXC alongside nginx
- Self-hosted services behind NPM: wiki, notes, RSS reader, cloud storage, dashboard

On paper, clean architecture. In practice, a recurring headache that took longer than it should have to diagnose correctly, partly because each fix introduced a new failure mode that wasn't visible until the previous one was resolved.

## Round One: NPM Had No SSL and the Tunnel Was Bypassing It Entirely

### The Symptoms

Hitting a subdomain from a LAN browser would sometimes work, sometimes return `ERR_SSL_UNRECOGNIZED_NAME_ALERT`, and sometimes show a Cloudflare 1033 tunnel error. Restarting things randomly would fix it temporarily. NPM was blamed. NPM was innocent.

### What Was Actually Wrong

Two problems compounding each other.

Every proxy host in NPM was set to HTTP Only. No certificates. NPM was terminating connections on port 80 and proxying to backends — fine for internal HTTP, but modern browsers are aggressive about HTTPS for domains they've seen before. Once a browser tries HTTPS and gets nothing, it starts caching that failure. The same URL would work in a fresh browser and fail in one that had cached the SSL error.

Browser HSTS makes this worse. Once a domain has been seen over HTTPS via Cloudflare, the browser caches the HSTS policy and insists on HTTPS everywhere — including on the LAN, where NPM had no cert to offer. That's why the same URL worked on one device and failed on another. It wasn't DNS. It wasn't the tunnel. It was the browser's memory of a previous HTTPS response.

The second problem: the `config.yml` for cloudflared pointed every subdomain directly at the backend service by IP and port. NPM was completely out of the picture for external traffic. SSL configuration in NPM was irrelevant for anything coming through the tunnel, access lists and future auth middleware would have to be maintained in two places, and if a backend went down the tunnel returned a 1033 with no useful information.

### The Fixes

**Wildcard cert via DNS challenge.** HTTP challenge won't validate a wildcard. Because the domain is on Cloudflare, use DNS challenge. NPM supports Cloudflare DNS challenge natively. Create a Cloudflare API token scoped to DNS Edit on your zone, paste it into NPM as `dns_cloudflare_api_token = YOUR_TOKEN`, and attach the resulting wildcard cert to every proxy host with Force SSL enabled.

**Route all tunnel traffic through NPM.** If your main domain serves static files directly from nginx on the same host as cloudflared, keep that — it's fine. Everything else should hit NPM. One entry point, consistent behavior, and any auth layer you add later covers everything at once without split configuration.

**Clear the HSTS cache if needed.** `chrome://net-internals/#hsts`, find the domain, delete it. The permanent fix is the cert — once NPM can serve HTTPS, the browser stops having a problem.

What the traffic path should look like:
```
Internet → Cloudflare → tunnel → nginx → NPM → backend
```

What it actually had:
```
Internet → Cloudflare → tunnel → nginx → backend  (NPM skipped entirely)
LAN: Browser → Pi-hole → NPM (no cert) → SSL error
```

Once the full path was on paper, the fix was obvious.

## Round Two: The Container Wouldn't Start

Months later. Woke up to Cloudflare 1033 errors across all tunneled services. The tunnel LXC had gone down and wouldn't come back up.

```
lxc_init: Failed to run lxc.hook.pre-start for container "CTID"
startup for container 'CTID' failed
```

No hook was registered in the container config. The error is misleading. Proxmox injects its own pre-start hook at runtime regardless of what's in the conf file — it doesn't appear in `/etc/pve/lxc/CTID.conf` and won't show up in a grep. The systemd journal gives you nothing useful either. The actual error requires a debug log:

```bash
lxc-start -n CTID -F -l DEBUG -o /tmp/lxc-debug.log 2>&1
cat /tmp/lxc-debug.log | grep -i "error\|fail\|resolv\|mount\|hook"
```

Output:
```
Script exec lxc-pve-prestart-hook produced output: close (rename) atomic file '/etc/resolv.conf' failed: Operation not permitted
error in setup task PVE::LXC::Setup::pre_start_hook
```

The Proxmox pre-start hook mounts the container's LVM volume, stages files into it, and writes `/etc/resolv.conf` before the container starts. It was getting blocked on the write.

### Dead Ends

The obvious guesses were wrong. Bind mount host paths existed and were mounted. The LVM volume was healthy. The staged rootfs mount point was clean. The hook is a Perl script — running it with `bash -x` produces syntax errors and tells you nothing useful. The debug log is the only diagnostic path.

### The Fix

Mounting the LVM volume manually and checking attributes:

```bash
mkdir -p /mnt/tmp-lxc
mount /dev/pve/vm-CTID-disk-0 /mnt/tmp-lxc
lsattr /mnt/tmp-lxc/etc/resolv.conf
```

```
----i---------e------- /mnt/tmp-lxc/etc/resolv.conf
```

The `i` flag. The file was set immutable at some point — probably during an earlier troubleshooting session that set it to prevent overwriting, then was never cleared. The hook can't rename-over an immutable file regardless of permissions or storage state.

```bash
chattr -i /mnt/tmp-lxc/etc/resolv.conf
umount /mnt/tmp-lxc
pct start CTID
```

Running.

## Round Three: Force SSL Broke External Access After the Container Came Back

Container up, tunnel running, services returning `ERR_TOO_MANY_REDIRECTS` on mobile data. LAN access worked fine.

The LAN vs. external split is the diagnostic tell. LAN traffic resolves via Pi-hole to NPM and never touches Cloudflare. External traffic goes through the tunnel. The problem was in the tunnel path specifically.

### Why Force SSL Was Enabled

This is worth explaining because it wasn't arbitrary. After the wildcard cert was in place, internal traffic on some browsers was hairpinning through the tunnel. Browser hits `service.yourdomain.tld`, Pi-hole doesn't have a local record for it, DNS falls through to Cloudflare, traffic leaves the LAN and comes back through the tunnel instead of resolving internally. Force SSL was part of the fix for that, along with adding Pi-hole DNS records pointing each subdomain to NPM directly.

Force SSL is correct for direct browser connections. It's wrong when the upstream is cloudflared.

### The Loop

cloudflared sends HTTP to NPM on port 80. NPM has Force SSL enabled and issues a 301 redirect to HTTPS. cloudflared follows the redirect. The request comes back through the tunnel as HTTP. Loop.

The fix is to make cloudflared speak HTTPS to NPM so NPM sees an already-secured connection and has nothing to redirect. But switching the tunnel config to HTTPS introduced a new problem.

### The SNI Problem

Without an explicit hostname in the TLS handshake, cloudflared connects to NPM's IP and port without SNI. NPM receives a TLS connection, doesn't know which virtual host is being requested, and rejects it:

```
tls: unrecognized name
```

This produces a 502 at the browser. The cloudflared logs show the real error, but the browser just shows 502. Without checking the container logs directly, this looks like a connectivity problem.

The fix requires two options together. `noTLSVerify` tells cloudflared not to validate NPM's certificate chain (it's a public Let's Encrypt cert on an internal IP — validation will fail). `originServerName` provides the hostname for the SNI field so NPM knows which virtual host to serve.

Working config for any subdomain routed through NPM:

```yaml
- hostname: subdomain.yourdomain.tld
  service: https://YOUR-NPM-IP:443
  originRequest:
    noTLSVerify: true
    originServerName: subdomain.yourdomain.tld
```

Subdomains served directly from nginx on the cloudflared host stay as HTTP and don't need this.

## The Full Picture

Five distinct problems across two sessions, none of them pointing cleanly at the others:

1. NPM had no SSL configured
2. The tunnel was bypassing NPM entirely
3. Browser HSTS caching inconsistencies across devices
4. Immutable flag on `/etc/resolv.conf` inside the container
5. Force SSL redirect loop with an SNI mismatch in the tunnel config

The diagnostic approach that worked each time was the same: map the full traffic path, identify which layer the failure was actually occurring at, and check the logs at that layer specifically rather than the ones that surface the symptom. The 1033 error isn't in the tunnel. The 502 isn't in NPM. The redirect loop isn't in the browser. Following the symptom to its source rather than stopping at the first plausible explanation is what eventually cleared all of it.

## Key Takeaways

- **Wildcard cert via DNS challenge** is the correct approach for `*.yourdomain.tld` in NPM. HTTP challenge won't validate a wildcard.
- **Route all tunnel traffic through NPM**, not directly to backends. NPM is your choke point — use it.
- **The Proxmox pre-start hook error is generic.** The real cause is in the debug log, not the systemd journal. `lxc-start -n CTID -F -l DEBUG -o /tmp/debug.log` is the command.
- **`lsattr` is not part of most people's troubleshooting reflex.** It should be. Immutable flags survive reboots and permission changes and produce "Operation not permitted" errors that look like everything except what they are.
- **cloudflared to NPM over HTTPS requires both `noTLSVerify` and `originServerName`.** The first skips cert validation. The second provides the SNI NPM needs to route the connection. Without both, you get a 502 and a TLS error in the cloudflared logs that doesn't obviously explain itself.
- **Trace the full traffic path** before blaming any single component. Symptoms at one layer often have root causes two layers away.

---

*Written with assistance from Claude (Anthropic) as part of an ongoing homelab documentation practice. The architecture decisions, the direction, and the opinions are mine. Claude helped build it faster and document it more clearly than I would have alone.*
