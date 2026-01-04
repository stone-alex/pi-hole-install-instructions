## Hardware

- CanaKit Raspberry Pi kit (Pi 3 fine) + short Ethernet cable.

## Imaging microSD

- Raspberry Pi Imager from raspberrypi.com/software
- Run it (chmod 755 if needed)
- Select Pi model, 64-bit OS, USB storage
- Hostname: e.g., raspberry (no spaces)
- Timezone/keyboard
- Username/password (strong!)
- Enable SSH (password auth)
- Skip Wi-Fi
- Write/verify


------------------------------
## Router

DHCP reservation for static IP (note MAC address)


------------------------------
## Install Pi-Hole

On Pi via terminal:
```
ssh admin@raspberry
```


```
sudo apt update && sudo apt full-upgrade -y

curl -sSL https://install.pi-hole.net | bash
```

Static IP: yes
Interface: eth0
Upstream: Quad9 filtered NCS DNSSEC
Default list: yes
Logging: off (SD card friendly)

Change admin password after.

------------------------------

## Install Unbound

```
sudo apt update
sudo apt install unbound

sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
```
Paste this config:

```
server:
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes
    do-ip6: no
    prefer-ip6: no
    harden-glue: yes

    harden-dnssec-stripped: yes
    use-caps-for-id: no
    edns-buffer-size: 1232
    prefetch: yes

    num-threads: 2

    so-rcvbuf: 1m
    chroot: ""

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: 10.1.1.0/24
    private-address: fd00::/8
    private-address: fe80::/10

    # Ensure no reverse queries to non-public IP ranges (RFC6303 4.2)
    private-address: 192.0.2.0/24
    private-address: 198.51.100.0/24
    private-address: 203.0.113.0/24
    private-address: 255.255.255.255/32
    private-address: 2001:db8::/32
```

Next run:
```
systemctl enable unbound.service
```

And:
```
systemctl start unbound.service
```

If ever need to restart:

```
systemctl restart unbound.service
```

At this point, DNS resolves via Pi-hole on Raspberry Pi. But not done yet. Now add domains to block.

Search "pi-hole block lists". Plenty out there. Good start in 2026: Hagezi's lists (highly recommended).

Link in description: https://github.com/hagezi/dns-blocklists

Login to Pi-hole admin, navigate to lists. Add one URL at a time. The more, the better.

When done, Tools > Update Gravity > Update. Don't leave page until done. Builds database of naughty servers to block. Not 100% protection from slop, but blocks 97% of trash: smut sites, malware, annoying ads, trackers, fingerprinters, and other junk unbecoming of your eyes.

And that's it! Enjoy more private, secure internet. No need for browser ad blockers, any device on home network gets filtered. Great parental control. Add domains manually under Domains. (Don't confuse with block lists. Those are curated files with multiple domains.)

Update lists often, rebuild Gravity database.

Enjoy.
