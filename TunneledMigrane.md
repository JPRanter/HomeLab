# Fixing HTTPS in a Homelab: Wildcard Certs, Cloudflare Tunnels, and Why It Was Never NPM's Fault

If you run Nginx Proxy Manager with a Cloudflare tunnel and keep getting SSL errors, 502s, or 1033 tunnel errors — this is probably your problem too.

## The Setup

A fairly common homelab stack:

- **Nginx Proxy Manager (NPM)** handling internal reverse proxy for all services
- **Pi-hole** for local DNS — all subdomains on a custom TLD resolve to NPM internally
- **Cloudflare Tunnel** (cloudflared) providing public access without port forwarding
- A handful of self-hosted services — wiki, notes, RSS reader, cloud storage — all sitting behind NPM

On paper, clean architecture. In practice, a recurring headache that took an embarrassingly long time to diagnose correctly.

## The Symptoms

Hitting a subdomain from a browser on the LAN would sometimes work, sometimes return `ERR_SSL_UNRECOGNIZED_NAME_ALERT`, and sometimes show a Cloudflare 1033 tunnel error. Restarting things randomly would fix it temporarily. NPM was blamed. NPM was innocent.

## What Was Actually Wrong

Three separate problems, all compounding each other.

### Problem 1: NPM Had No SSL Configured

Every proxy host in NPM was set to HTTP Only. No certificates. NPM was terminating connections on port 80 and proxying to backends — fine for internal HTTP, but the moment a browser decided to try HTTPS (which modern browsers do aggressively, especially for domains they've seen before), NPM had nothing to offer.

The fix: issue a wildcard Let's Encrypt certificate for your domain and attach it to every proxy host with Force SSL enabled.

Because the domain is on Cloudflare, HTTP challenge won't work for a wildcard — you need DNS challenge. NPM supports Cloudflare DNS challenge natively. Create a Cloudflare API token scoped to DNS Edit on your zone, paste it into NPM in the format `dns_cloudflare_api_token = YOUR_TOKEN`, and NPM handles the rest. The cert issues in under a minute.

### Problem 2: The Cloudflare Tunnel Was Bypassing NPM Entirely

The `config.yml` for cloudflared pointed every subdomain directly at the backend service — IP and port, no NPM in the middle. NPM was completely out of the picture for external traffic. This meant:

- SSL configuration in NPM was irrelevant for anything coming through the tunnel
- Access lists and future auth middleware would have to be configured in two places
- If a backend went down, the tunnel got nothing and returned a 1033

The fix: route everything through NPM instead of directly to backends.

If your main domain serves static files directly from nginx on the same host, keep that — it's fine. Everything else should hit NPM, which handles routing to the correct backend. One entry point, consistent behavior, and any auth layer you add later drops in at NPM and covers everything at once.

### Problem 3: Browser HSTS Cache

Once a browser has seen a domain served over HTTPS via Cloudflare, it caches HSTS and will insist on HTTPS for that domain everywhere — including when you're on the LAN and DNS is resolving to NPM's local IP. If NPM has no cert, the browser gets an SSL error even though the internal path is perfectly fine.

This is why the same URL worked in one browser and failed in another. The browser that worked hadn't cached HSTS for that domain yet.

The fix is the wildcard cert — once NPM can serve HTTPS, this stops being a problem. If you need to clear the cache manually in the meantime: `chrome://net-internals/#hsts`, find the domain, delete it.

## Why It Took So Long to See

Each symptom pointed somewhere else. The 1033 error looks like a tunnel problem. The SSL error looks like an NPM problem. The inconsistency across browsers looks like a DNS problem. None of them pointed at the real root cause.

The connection only became obvious when mapping the full traffic path end to end.

What it should look like:
```
Internet → Cloudflare → tunnel → nginx → NPM → backend
```

What the config actually had:
```
Internet → Cloudflare → tunnel → nginx → backend  (NPM skipped entirely)
```

And internally:
```
Browser → Pi-hole → NPM (no cert) → SSL error
```

Once the full path was on paper, the fix was straightforward.

## The Result

All subdomains now work correctly over HTTPS both internally and externally. Internal traffic resolves via Pi-hole to NPM, which terminates SSL with the wildcard cert and proxies to the backend. External traffic goes through the Cloudflare tunnel to NPM via the same path. One cert, one proxy, consistent behavior everywhere.

When an auth layer gets added, it drops in at NPM and covers everything in one place — no split configuration, no per-service middleware to maintain in two locations.

## Key Takeaways

- **Wildcard cert via DNS challenge** is the correct approach for `*.yourdomain.tld` in NPM. HTTP challenge won't validate a wildcard.
- **Route all tunnel traffic through NPM**, not directly to backends. NPM is your choke point — use it.
- **Browser HSTS is real** and will cause confusing inconsistencies when SSL isn't configured properly. The fix is proper SSL, not cache clearing.
- **Trace the full traffic path** before blaming any single component. Symptoms at one layer often have root causes two layers away.

---

*Written with assistance from Claude (Anthropic) as part of an ongoing homelab documentation practice.*
