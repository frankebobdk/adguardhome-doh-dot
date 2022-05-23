# Welcome to AdGuardHome DoH/DoT !

## 💻 Introduction

- Official **AdGuardHome Docker** with both **DoH** (DNS over HTTPS) and **DoT** (DNS over TLS) clients.
Don't browse the Internet insecurely by sending your DNS requests in clear text !
- Special thanks to [Trinib](https://github.com/trinib/AdGuard-WireGuard-Unbound-Cloudflare) for his amazing work on [AdGuard-WireGuard-Unbound-Cloudflare](https://github.com/trinib/AdGuard-WireGuard-Unbound-Cloudflare/blob/main/README.md) .
- Image built for Raspberry Pi (arm32/v7).

## 🚀 Installation

**_Don't forget to replace `<path_to_data>` to your docker data directory._**

**1. Clone the repository**

`git clone https://github.com/oijkn/adguardhome-doh-dot.git <path_to_data>`

**2. Edit docker-compose.yml file to fit your needs**

```dockerfile
version: "2"                                                           # Docker Compose version for Portainer

services:
  adguardhome:
    image: oijkn/adguardhome-doh-dot:latest
    container_name: adguardhome
    hostname: rpi-adguard
    environment:
      - PUID=1000                                                      # User ID (UID)
      - PGID=100                                                       # Group ID (GID)
      - TZ=Europe/Paris                                                # Timezone
      - LANG=fr_FR.UTF8                                                # Language
      - LANGUAGE=fr_FR.UTF8                                            # Language (same as LANG)
    tmpfs:
      - /run
      - /run/lock
      - /tmp
#    labels:
#      - "com.centurylinklabs.watchtower.enable=true"                  # Watchtower (auto update)
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
      - <path_to_data>/adguardhome/conf:/opt/AdGuardHome/conf          # Configure '<path_to_data>' to your needs
      - <path_to_data>/adguardhome/work:/opt/AdGuardHome/work          # Configure '<path_to_data>' to your needs
      - <path_to_data>/unbound/root.hints:/var/lib/unbound/root.hints  # Configure '<path_to_data>' to your needs
    cap_add:
      - NET_ADMIN
    networks:
      macvlan0:
        ipv4_address: 192.168.1.110                                    # IP of the container for AdGuardHome, configure it to your needs
    restart: unless-stopped

  cron:
    image: alpine:latest
    container_name: cron
    hostname: rpi-cron
    command: crond -f -d 8
    depends_on:
      - adguardhome
    volumes:
      - <path_to_data>/crontab/root:/etc/crontabs/root:z               # Configure '<path_to_data>' to your needs
      - <path_to_data>/unbound/root.hints:/tmp/unbound/root.hints      # Configure '<path_to_data>' to your needs
    restart: unless-stopped

networks:
  macvlan0:
    driver: macvlan
    driver_opts:
      parent: eth0                                                     # Parent interface, configure it depending on your interface name
    ipam:
      config:
        - subnet: 192.168.1.0/24                                       # Subnet of the container
          gateway: 192.168.1.1                                         # Gateway of the network
          ip_range: 192.168.1.100/28                                   # Usable Host IP Range: 192.168.1.97 - 192.168.1.110
          aux_addresses:
            rpi-srv: 192.168.1.100                                     # Reserved for RPi Server (IP of the host)
```

**3. Configure the host**

- Before running our DNS resolvers, it is a good idea to turn off [systemd-resolved](https://www.freedesktop.org/software/systemd/man/systemd-resolved.service.html).

  Edit the file `/etc/systemd/resolved.conf` as below :

```toml
[Resolve]
#DNS=
#FallbackDNS=
#Domains=
#LLMNR=yes
#MulticastDNS=yes
#DNSSEC=allow-downgrade
#DNSOverTLS=no
#Cache=yes
DNSStubListener=no        # Change this value to 'no' and uncomment
#ReadEtcHosts=yes

```

- Then, we need to configure the DNS resolvers.

  Edit the file `/etc/resolv.conf` as below :

```
nameserver 192.168.1.110  # IP of the container for AdGuardHome, configure it to your needs
```


## ☕ How to use it ?

- After configuring the docker-compose.yml file, you have to start the containers with: `docker-compose up -d`
- Then, you can access the AdGuardHome web interface at: `http://<IP_of_the_container_for_AdGuardHome>:3000/`
- `IMPORTANT`: In Listen Interfaces option choose `eth0` (or another name, it depends on your system) and select next
- Set up `username` & `password` and then login admin panel (port :80)
- `IMPORTANT`: In general settings, set "Query logs retention" to 24 hours. (I read that for some people logs fill up which slows down Pi and needing a reboot)
- In AdGuard homepage under settings, select "DNS settings"
  - Delete everything from both _**Upstream**_ and _**Bootstrap DNS**_ server options and add the following for:
  - DNS over TLS (Unbound) : `127.0.0.1:53`
  - DNS over HTTPS/Oblivious DNS over HTTPS :
     - `127.0.0.1:5053` (Cloudflared tunnel)
  - TLS forwarder (Stubby) : `127.0.0.1:8053` 
- `IMPORTANT:` Check "<a href="https://adguard.com/en/blog/in-depth-review-adguard-home.html#dns"><b>Parallel Request</b></a>" option for DNS resolvers to work simultaneously.
- Then in DNS setting look for DNS cache configuration section and set cache size to 0 (caching is already handled by Unbound) and click apply.
- Click apply and test upstreams

## 📝 Notes

- DoH service (Cloudflared) runs at 127.0.0.1#853.
- DoT service (Stubby) runs at 127.0.0.1#8053.
- DoH & DoT Uses cloudflare (1.1.1.1 / 1.0.0.1) by default

## 📫 Credits

- Trinib for the global configuration [AdGuard-WireGuard-Unbound-Cloudflare](https://github.com/trinib/AdGuard-WireGuard-Unbound-Cloudflare)
- [Cloudflared](https://developers.cloudflare.com/)
- [DNS Privacy Stubby resolver](https://github.com/getdnsapi/stubby)
- [Unbound DNS](https://unbound.net/)
- [AdGuardHome](https://github.com/AdguardTeam/AdGuardHome/blob/master/README.md)

## ✍️ User Feedback

If you have any problems with or questions about this image, please contact me through a [GitHub issue](https://github.com/oijkn/adguardhome-doh-dot/issues).