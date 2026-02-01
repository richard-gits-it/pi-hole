# Pi-hole Network-Wide Ad Blocker & DNS Filter

A Pi-hole is a network-wide ad blocker and DNS filter that provides protection for all devices on a network. I decided to build this Pi-hole to block advertisements and malicious domains across my entire home network without needing to configure individual devices. Through this project, I gained hands-on experience with DNS, Linux system administration, LXC containerization, and network integration.

<img width="4484" height="3484" alt="image" src="https://github.com/user-attachments/assets/5b9af845-13e6-455f-a72f-4d5c2af6bcdf" />

## Hardware & Platform
- Proxmox VE Hypervisor
- 500GB SSD (Proxmox OS & VM storage)
- 1TB HDD (ISO storage & backups)
- Debian 12 LXC Container

## Installation

The first step was downloading the Debian 12 LXC template in Proxmox:

```bash
pveam update
pveam download local debian-12-standard_12.12-1_amd64.tar.zst
```

This updates the available templates list and downloads the latest Debian 12 container template.

I then created an LXC container with the following specifications:
- **CT ID:** 100
- **Hostname:** pihole
- **Memory:** 512MB
- **CPU:** 1 core
- **Storage:** 8GB
- **Network:** Static IP (192.168.1.2/24)

After the container was created, I accessed the console and updated the system:

```bash
apt update
apt upgrade -y
apt install -y curl wget sudo nano
```

These commands ensure all packages are up to date and install essential tools needed for Pi-hole installation.

Next, I ran the Pi-hole installation script:

```bash
curl -sSL https://install.pi-hole.net | bash
```

**Curl** is used to fetch the installer script from the Pi-hole server. The flags mean:
- `-s` = silent mode (no progress bar)
- `-S` = show errors only
- `-L` = follow redirects

**Bash** receives the script from curl and executes it line by line.

After running this command, an installation wizard appeared. This wizard allowed me to customize my Pi-hole settings including:
- Upstream DNS provider (I chose Cloudflare for privacy)
- Blocklists (enabled default lists)
- Web admin interface (enabled)
- Query logging (enabled for monitoring)

Upon completion, the installer provided my Pi-hole's admin password and confirmed the web interface address: `http://192.168.1.2/admin`
<img width="558" height="357" alt="image" src="https://github.com/user-attachments/assets/15bdba7b-b78e-4a66-987f-e579db769a87" />

## Router Configuration

In my ASUS router interface, I navigated to:
**Advanced Settings → LAN → DHCP Server**

I configured the DNS settings to point to my Pi-hole:
- **DNS Server 1:** 192.168.1.2 (Pi-hole IP)
- **DNS Server 2:** (left blank)
- **Advertise router's IP in addition to user-specified DNS:** No
<img width="756" height="189" alt="image" src="https://github.com/user-attachments/assets/b7c900be-8f7e-4775-98bc-5611fcd187a0" />

This configuration ensures all devices on my network automatically receive Pi-hole as their DNS server through DHCP, eliminating the need to manually configure each device.

To verify my Pi-hole IP address, I used:

```bash
ip addr show eth0
```
<img width="964" height="142" alt="image" src="https://github.com/user-attachments/assets/63688fbc-f574-4427-a45c-3a72a512950a" />

This displays the network interface configuration and confirms the static IP address assigned to the container.

After saving the router settings, I rebooted the router to apply the changes. Once devices renewed their DHCP leases, they automatically started using Pi-hole for DNS resolution.

## Blocklist Configuration

I enhanced Pi-hole's blocking capabilities by adding additional blocklists targeting:
- Malware domains
- Phishing sites
- Tracking and telemetry
- Cryptomining scripts
- Smart TV tracking

To add blocklists, I accessed the Pi-hole admin interface and navigated to:
**Adlists → Add new adlist**

I added curated lists from sources like:
- AdGuard DNS Filter
- Phishing Army
- URLhaus Malware Database
- Windows Telemetry Blocker

After adding the lists, I updated Gravity to apply the changes:

```bash
pihole -g
```

This command downloads and compiles all blocklists into Pi-hole's blocking database. My final blocklist count reached over 600,000 domains.

## Verification & Testing

To verify Pi-hole was working correctly, I ran several tests from my Windows machine:

```powershell
ipconfig /flushdns
nslookup google.com
```

The `nslookup` command confirmed that DNS queries were being resolved by Pi-hole (192.168.1.2) instead of my ISP's DNS servers.
<img width="391" height="172" alt="image" src="https://github.com/user-attachments/assets/910139fb-2671-4e76-8a24-9a189592fc8a" />

I also tested the special Pi-hole domain:

```
http://pi.hole/admin
```

This successfully loaded the Pi-hole dashboard, confirming proper DNS integration.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/83fb24bd-7ab3-4edc-b829-04589ef72c1e" />

Additionally, I visited ad-blocking test sites:
- canyoublockit.com
- adblock-tester.com
<img width="1920" height="947" alt="image" src="https://github.com/user-attachments/assets/ef27c9c0-4150-40a6-9898-230c066ef55b" />

Both sites confirmed high blocking rates, validating that Pi-hole was successfully filtering advertisements and trackers.

## Results

After 24 hours of operation, my Pi-hole dashboard showed:
- **Total Queries:** ~10,000+
- **Queries Blocked:** 2,500+ (approximately 26%)
- **Domains on Blocklist:** 300,000+
- **Active Clients:** 19 devices
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e68145eb-9482-400f-a27b-d0cc638ac4cc" />


I immediately noticed several improvements:
- No more pop-up advertisements on websites
- Significantly faster page load times (ads don't load)
- Reduced mobile data usage when on home network
- Elimination of tracking across all devices

The Pi-hole query log revealed numerous blocked domains including ad networks, analytics trackers, telemetry services, and known malicious domains that would have otherwise been accessed by devices on my network.

## Key Takeaways

Through this project, I strengthened my skills in several key areas:

**Virtualization & Containerization:** Gained hands-on experience with Proxmox VE and LXC containers, understanding the benefits of lightweight virtualization for network services.

**Linux System Administration:** Improved proficiency with Debian Linux, package management, systemd services, and command-line tools for troubleshooting and configuration.

**DNS & Network Fundamentals:** Deepened understanding of DNS resolution, DHCP integration, static IP configuration, and how DNS-level filtering provides network-wide protection.

**Network Security:** Learned how DNS filtering can block malicious domains, prevent tracking, and reduce attack surface across an entire network without endpoint software.

**Router Integration:** Gained experience configuring enterprise-grade network appliances to work with custom DNS infrastructure.

I was able to see firsthand how DNS plays a critical role in both network performance and security. Understanding how Pi-hole intercepts DNS queries and blocks unwanted domains before they reach client devices demonstrates the importance of DNS filtering in modern network architecture. This project provided valuable experience that directly applies to enterprise network security, IT infrastructure management, and cybersecurity operations.

## Future Enhancements

Potential improvements for this project include:
- Implementing DNS-over-HTTPS (DoH) for encrypted upstream queries
- Setting up a secondary Pi-hole instance for redundancy
- Configuring conditional forwarding for local domain resolution
- Integrating VPN access to use Pi-hole when away from home
- Implementing automated backup solutions for Pi-hole configuration

---

**Technologies Used:** Pi-hole, Proxmox VE, LXC Containers, Debian Linux, DNS, DHCP, Network Security
