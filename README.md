# Wazuh Decoder and Rules for MikroTik RouterOS

This directory contains Wazuh decoders and rules for monitoring MikroTik RouterOS devices.

## Files

- `decoder-mikrotik.xml` - Decoders for parsing MikroTik syslog messages
- `rules-mikrotik.xml` - Rules for detecting security events
- `README.md` - This file

## Installation

### 1. Copy Files

```bash
# Copy decoder
sudo cp decoder-mikrotik.xml /var/ossec/etc/decoders/

# Copy rules
sudo cp rules-mikrotik.xml /var/ossec/etc/rules/
```

### 2. Update ossec.conf

Edit `/var/ossec/etc/ossec.conf` and ensure the files are included:

```xml
<ossec_config>
  <!-- Include decoder -->
  <decoder_dir>/etc/decoders</decoder_dir>
  
  <!-- Include rules -->
  <rules>
    <rule_include>rules-mikrotik.xml</rule_include>
  </rules>
  
  <!-- Configure syslog input for MikroTik -->
  <remote>
    <connection>syslog</connection>
    <port>514</port>
    <protocol>udp</protocol>
  </remote>
</ossec_config>
```

### 3. Restart Wazuh

```bash
sudo systemctl restart wazuh-manager
```

## MikroTik Configuration

### Enable Syslog on MikroTik

1. **Using Winbox/Webfig:**
   - Go to `System` → `Logging`
   - Go to `Actions` tab
   - Add a new remote action:
     - Name: `wazuh`
     - Type: `remote`
     - Remote Address: `YOUR_WAZUH_SERVER_IP`
     - Port: `514`
     - Syslog Facility: `daemon` (or your preference)
   - Go to `Rules` tab
   - Add rules for the events you want to monitor, selecting the `wazuh` action

2. **Using CLI:**

```bash
# Set up remote syslog action
/system logging action add name=wazuh target=remote remote=YOUR_WAZUH_SERVER_IP remote-port=514

# Configure logging rules (add topics as needed)
/system logging add topics=firewall action=wazuh prefix=mikrotik
/system logging add topics=system action=wazuh prefix=mikrotik
/system logging add topics=error action=wazuh prefix=mikrotik
/system logging add topics=critical action=wazuh prefix=mikrotik
/system logging add topics=account action=wazuh prefix=mikrotik
/system logging add topics=interface action=wazuh prefix=mikrotik
/system logging add topics=ppp action=wazuh prefix=mikrotik
/system logging add topics=wireguard action=wazuh prefix=mikrotik
/system logging add topics=hotspot action=wazuh prefix=mikrotik

# For VPN monitoring
/system logging add topics=l2tp action=wazuh prefix=mikrotik
/system logging add topics=ovpn action=wazuh prefix=mikrotik
/system logging add topics=ipsec action=wazuh prefix=mikrotik
```

### Recommended Topics to Monitor

| Topic | Description |
|-------|-------------|
| `firewall` | Firewall hits, drops, NAT |
| `system` | System events, startup |
| `error` | Error messages |
| `critical` | Critical events |
| `account` | Login/logout events |
| `interface` | Interface state changes |
| `ppp` | PPP connections |
| `wireguard` | WireGuard VPN events |
| `ovpn` | OpenVPN events |
| `ipsec` | IPsec/L2TP events |
| `hotspot` | Hotspot user events |
| `dhcp` | DHCP assignments |
| `script` | Script executions |
| `web` | Webfig/REST API access |
| `winbox` | Winbox connections |

## Log Format Examples

### Authentication Logs
```
system,info,account user admin logged in from 192.168.1.100 via winbox
system,error,account login failure for user admin from 192.168.1.100 via ssh
system,info,account user admin logged out from 192.168.1.100 via winbox
```

### Firewall Logs
```
firewall,info forward: in:ether1 out:ether2, proto TCP (SYN), 192.168.1.10:12345->10.0.0.1:80, NAT 192.168.1.10:12345->(10.0.0.1:80)->10.0.0.1:80, len 60
firewall,info drop: in:ether1 out:bridge1, proto TCP (SYN), 192.168.1.50:54321->192.168.1.1:22, len 60
firewall,info invalid: in:ether1 out:bridge1, proto TCP, 192.168.1.10:80->192.168.1.1:12345, len 40
```

### VPN Logs
```
ppp,info,ppp PPP connected for user john from 203.0.113.10
ppp,info,ppp PPP disconnected for user john from 203.0.113.10
ovpn,info OVPN client connected: john from 198.51.100.5
ipsec,info IPsec SA established: 192.168.1.0/24[0] 192.168.2.0/24[0]
```

## Rules Reference

### Authentication (11xxxx)

| Rule ID | Level | Description |
|---------|-------|-------------|
| 110001 | 3 | Successful user login |
| 110002 | 5 | Failed login attempt |
| 110003 | 10 | Multiple failed logins (5 in 120s) |
| 110004 | 3 | User logged out |

### Firewall (1101xx)

| Rule ID | Level | Description |
|---------|-------|-------------|
| 110101 | 4 | Firewall dropped connection |
| 110102 | 7 | Multiple drops from same source (10 in 60s) |
| 110103 | 4 | Invalid packet dropped |
| 110104 | 8 | Possible port scan detected (15 SYN in 60s) |

### System (1102xx)

| Rule ID | Level | Description |
|---------|-------|-------------|
| 110201 | 4 | Router has started |
| 110202 | 6 | System error detected |
| 110203 | 9 | Critical system error |

### Configuration (1103xx)

| Rule ID | Level | Description |
|---------|-------|-------------|
| 110301 | 5 | Configuration was changed |
| 110302 | 4 | Script executed |
| 110303 | 4 | Backup created |

### Network (1104xx)

| Rule ID | Level | Description |
|---------|-------|-------------|
| 110401 | 5 | Interface went down |
| 110402 | 3 | Interface came up |

### VPN (1105xx)

| Rule ID | Level | Description |
|---------|-------|-------------|
| 110501 | 3 | PPP connection established |
| 110502 | 3 | PPP connection terminated |
| 110503 | 3 | IPsec SA established |
| 110504 | 3 | OpenVPN client connected |
| 110505 | 3 | WireGuard peer connected |

### Hotspot (1106xx)

| Rule ID | Level | Description |
|---------|-------|-------------|
| 110601 | 3 | Hotspot user logged in |
| 110602 | 3 | Hotspot user logged out |

### DHCP (1107xx)

| Rule ID | Level | Description |
|---------|-------|-------------|
| 110701 | 1 | DHCP IP assigned |
| 110702 | 1 | DHCP IP deassigned |

## Testing

### Verify Decoder

```bash
# Test with sample log
sudo /var/ossec/bin/wazuh-logtest
```

Enter a sample log line:
```
system,info,account user admin logged in from 192.168.1.100 via winbox
```

Expected output should show the decoder extracting fields.

### Verify Rules

Check if rules are loaded:
```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```

## Troubleshooting

### Logs not appearing

1. Check if syslog is listening on Wazuh server:
   ```bash
   sudo netstat -uln | grep 514
   ```

2. Check firewall rules:
   ```bash
   sudo iptables -L | grep 514
   ```

3. Check Wazuh logs:
   ```bash
   sudo tail -f /var/ossec/logs/ossec.log
   ```

### Decoder not matching

1. Test decoder manually:
   ```bash
   sudo /var/ossec/bin/wazuh-logtest -v
   ```

2. Check decoder is loaded:
   ```bash
   sudo /var/ossec/bin/wazuh-dbd -D
   ```

## Contributing

Feel free to submit issues or pull requests to improve these decoders and rules.

## License

This project is provided as-is for use with Wazuh SIEM platform.
