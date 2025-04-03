# Selfhosted headscale, derp node with domain and https.

**NOTE: Need have a domain, this setup does not use self-signed cert.**

# Prerequisite

1. Your own a domain, e.g `abc.xyz`

2. You have VPS and it has **public IP**, e.g AWS EC2 instance or others

3. OS Ubuntu/Debian, open ports **80, 443, 3478/udp, 3478/tcp**

4. DNS for `derp.abc.xyz`, `headscale.abc.xyz`

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

In this example, we have linux device A at home (CIDR 10.0.0.0/24), we want access home network while outside,
device A perform as subnet router.

[Subnet routers] (https://tailscale.com/kb/1019/subnets)

## Enable IP forwarding
```
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p /etc/sysctl.conf
```

## 
`sudo tailscale up --login-server=https://headscale.abc.xyz --advertise-routes=10.0.0.0/24`

If you have more subnet, command like below:

`sudo tailscale up --login-server=https://headscale.abc.xyz --advertise-routes=10.0.0.0/24,198.51.100.0/24`

Approve subnet in headscale ui.

## Firewall

```
sudo apt install iptables-persistent
sudo netfilter-persistent save

# Basic rule
# Allowing Loopback Connections
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT

# Allowing Established and Related Incoming Connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allowing Established Outgoing Connections
sudo iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED -j ACCEPT

# Dropping Invalid Packets
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# Allowing All Incoming SSH
sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT

# Allowing Outgoing SSH
sudo iptables -A OUTPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT

# For tailscale
read -p "eth interface, e.g eth0:" WAN_IF
TS_IF=tailscale0

sudo iptables -t nat -A POSTROUTING -o $WAN_IF -j MASQUERADE
sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i $TS_IF -o $WAN_IF -j ACCEPT

# optional
sudo iptables -I FORWARD -i $WAN_IF -j ACCEPT
sudo iptables -I FORWARD -o $WAN_IF -j ACCEPT
sudo iptables -I FORWARD -i $TS_IF -j ACCEPT
sudo iptables -I FORWARD -o $TS_IF -j ACCEPT
sudo iptables -t nat -I POSTROUTING -o $TS_IF -j MASQUERADE

# Save
sudo netfilter-persistent save
```

## Use your subnet routes from other devices
Android, iOS, macOS, tvOS, and Windows automatically pick up your new subnet routes.

By default, Linux devices only discover Tailscale IP addresses. 

To enable automatic discovery of new subnet routes on Linux devices, use the --accept-routes flag when you start Tailscale:

`sudo tailscale up --login-server=https://headscale.abc.xyz --accept-routes`
