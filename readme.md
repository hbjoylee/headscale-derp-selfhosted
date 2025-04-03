# Selfhosted headscale, derp node with domain and https.

**NOTE: Need have a domain, this setup does not self signed cert.**

# Prerequisite

1. Your own a domain, e.g abc.xyz

2. You have VPS and it has static IP, e.g AWS EC2 instance or others

3. OS Ubuntu/Debian, open ports 80, 443, 3478/udp, 3478/tcp

4. DNS for derp.abc.xyz, headscale.abc.xyz

5. Install Docker Engine


# Create docker network

`docker network create caddy`

# DERP node

DERP server only open http port and verify clients to block unauthorized users

## Install tailscale

`curl -fsSL https://tailscale.com/install.sh | sh`

## Build DERP docker image and start it up

```
cd derp
./build.sh
docker compose up -d
```

# Headscale

# Caddy
