# Akamai-network-engineering-project
# Building-Multizone-secure-firewall

# Enterprise Multi-Zone Secure Firewall Architecture on Red Hat Linux

## Overview

This project demonstrates how to design and implement an enterprise-grade, multi-zone secure firewall architecture 
using Red Hat Enterprise Linux (RHEL). It uses firewalld, nftables/iptables concepts, and network segmentation to protect internal infrastructure.

---

## Architecture Design
### Zones

* **External Zone (Untrusted)**: Internet-facing interface
* **DMZ Zone (Semi-trusted)**: Public-facing services (web, mail, etc.)
* **Internal Zone (Trusted)**: Corporate network
* **Management Zone (Highly restricted)**: Admin access only

### Example Network Layout

* External: 192.0.2.0/24
* DMZ: 172.16.0.0/24
* Internal: 10.0.0.0/24
* Management: 192.168.10.0/24

---

## Prerequisites

* Red Hat Enterprise Linux 8/9
* Root or sudo privileges
* 2+ network interfaces (or virtual interfaces)

Install required packages:

```bash
sudo dnf install -y firewalld iproute
```

Enable firewall:

```bash
sudo systemctl enable --now firewalld
```

---

## Step 1: Configure Network Interfaces

Assign interfaces to zones:

```bash
nmcli connection modify eth0 connection.zone external
nmcli connection modify eth1 connection.zone dmz
nmcli connection modify eth2 connection.zone internal
nmcli connection modify eth3 connection.zone trusted
```

Reload:

```bash
nmcli connection reload
```

---

## Step 2: Configure Zones

List zones:

```bash
firewall-cmd --get-zones
```

Set default zone:

```bash
firewall-cmd --set-default-zone=external
```

---

## Step 3: Define Firewall Rules

### External Zone

Allow only necessary inbound traffic:

```bash
firewall-cmd --zone=external --add-service=ssh --permanent
firewall-cmd --zone=external --add-service=http --permanent
firewall-cmd --zone=external --add-service=https --permanent
```

### DMZ Zone

Allow public services:

```bash
firewall-cmd --zone=dmz --add-service=http --permanent
firewall-cmd --zone=dmz --add-service=https --permanent
```

### Internal Zone

Allow internal communication:

```bash
firewall-cmd --zone=internal --add-service=ssh --permanent
firewall-cmd --zone=internal --add-service=samba --permanent
```

### Management Zone

Restrict strictly:

```bash
firewall-cmd --zone=trusted --add-source=192.168.10.0/24 --permanent
firewall-cmd --zone=trusted --add-service=ssh --permanent
```

Apply changes:

```bash
firewall-cmd --reload
```

---

## Step 4: Enable NAT (Masquerading)

Allow internal users to access internet:

```bash
firewall-cmd --zone=external --add-masquerade --permanent
firewall-cmd --reload
```

---

## Step 5: Configure Port Forwarding (DNAT)

Forward external HTTP traffic to DMZ web server:

```bash
firewall-cmd --zone=external --add-forward-port=port=80:proto=tcp:toaddr=172.16.0.10 --permanent
firewall-cmd --reload
```

---

## Step 6: Inter-Zone Policies

Block direct access from external to internal:

```bash
firewall-cmd --zone=external --add-rich-rule='rule family="ipv4" destination address="10.0.0.0/24" reject' --permanent
```

Allow internal to DMZ:

```bash
firewall-cmd --zone=internal --add-rich-rule='rule family="ipv4" destination address="172.16.0.0/24" accept' --permanent
```

Reload:

```bash
firewall-cmd --reload
```

---

## Step 7: Logging and Monitoring

Enable logging:

```bash
firewall-cmd --set-log-denied=all
```

View logs:

```bash
journalctl -xe | grep firewalld
```
---

## Step 8: Testing

Test connectivity:

```bash
ping 8.8.8.8
curl http://example.com
```

Test port exposure:

```bash
nmap -p 80,443 <public-ip>
```

---

## Project Structure

```
firewall-project/
├── README.md
├── scripts/
│   ├── setup.sh
│   ├── rules.sh
├── configs/
│   ├── firewalld-zones.xml
└── docs/
    ├── architecture-diagram.md
```

---

## Example Setup Script

```bash
#!/bin/bash

# Enable firewalld
systemctl enable --now firewalld

# Assign zones
nmcli connection modify eth0 connection.zone external
nmcli connection modify eth1 connection.zone dmz
nmcli connection modify eth2 connection.zone internal

# Add rules
firewall-cmd --zone=external --add-service=http --permanent
firewall-cmd --zone=external --add-service=https --permanent

# Enable NAT
firewall-cmd --zone=external --add-masquerade --permanent

firewall-cmd --reload
```


## Security Best Practices

* Disable unused services
* Use SSH key authentication
* Restrict management zone access
* Regularly update system
* Implement intrusion detection (e.g., fail2ban)

---

## Future Enhancements

* Integrate SELinux policies
* Add IDS/IPS (Snort/Suricata)
* Automate via Ansible
* Centralized logging (ELK stack)

---

## License

MIT License
