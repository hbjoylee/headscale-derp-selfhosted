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

# Caddy

DERP, Headscale and Headscale all open 8080 port in this setup, you need change port in Caddyfile if use different ports.

1. Update url in caddy/Caddyfile

2. Start caddy `docker compose up -d`


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
## Check DERP in browser

Open https://derp.abc.xyz, you should see status page.


# Headscale and Headscale UI

Update `server_url` in `container-config/config.yaml`

`server_url: https://headscale.abc.xyz`

Update `hostname` and `ipv4` in `container-config/self-derp.yaml`

```
hostname: derp.abc.xyz
ipv4: YOUR_PUBLIC_IP
```
## Start headscale and UI

```
cd headscale/
docker compose up -d
```

## Generate API key
`docker exec -it headscale headscale apikeys create --expiration 9999d`

## Login to web ui

Open `https://headscale.abc.xyz/web` in browser, paste api key to **Headsscale API Key** field, then click **Test Server Settings**, if you see green tick, it means you connected to headscale backend successfully.

## Create user in Headscale

Click **User View** in left panel and create user, e.g `hcuser`, once you have new device join, you need assgin user to this device.
 
## Add device

Install tailscale on device then execute command below to join 

`sudo tailscale up  --login-server=https://headscale.abc.xyz`

it retuns link as below:

`https://headscale.abc.xyz/register/nodekey:xxxyyyzzzooommmppp`

You only need copy `xxxyyyzzzooommmppp` and paste to headscale web ui

# Route, forwarding and firewall
