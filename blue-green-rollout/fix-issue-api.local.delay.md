🎯 Root cause: The delay happens because macOS tries IPv6 (::1) first, times out, then falls back to IPv4 (127.0.0.1). This is a common Docker Desktop + macOS + Ingress issue.

## Quick Fix - Force IPv4:
```bash
# Use IPv4 explicitly
curl -4 http://api.local
```

## Permanent Fix - Update /etc/hosts:
```bash
sudo nano /etc/hosts

# Change from:
127.0.0.1 api.local

# To (add both IPv4 and IPv6):
127.0.0.1 api.local
::1 api.local
```
