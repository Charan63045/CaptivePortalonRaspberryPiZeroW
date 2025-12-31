# 🌐 Auto-Captive Portal for Raspberry Pi Zero W

An intelligent, self-managing WiFi configuration system with automatic failover, hardware button control, and secure credential management. Perfect for headless Raspberry Pi deployments in IoT applications.

## 📋 Overview

This captive portal system automatically activates when internet connectivity is lost, providing a web-based interface for WiFi configuration. Built for Raspberry Pi Zero W/3B+, it features encrypted credential storage, automatic network recovery, and manual activation via hardware button.

## ✨ Key Features

### 🔄 **Automatic Network Management**
- Monitors internet connectivity every 5 seconds
- Auto-activates captive portal after 7 seconds of no internet
- Attempts saved networks on boot and connection loss
- Automatically shuts down portal when connection restored

### 🔐 **Secure Credential Storage**
- AES-256 encryption using Fernet (cryptography library)
- Three save modes: encrypted, temporary, or no-save
- Quick-connect to previously saved networks
- Encrypted credentials stored in `/etc/mypi/`

### 🎛️ **Hardware Button Control**
- GPIO pin 18 for manual activation
- Hold button 3+ seconds to force configuration mode
- Works even when internet is available
- Internal pull-up resistor configuration

### 🎨 **Modern Web Interface**
- Clean, responsive Material Design UI
- Real-time network scanning with signal strength indicators
- Security type badges (Open/WPA/WPA2/WPA3)
- Frequency band detection (2.4GHz/5GHz)
- Mobile-optimized interface

### 🚀 **Production-Ready Features**
- Systemd service integration
- Auto-start on boot
- Comprehensive logging
- Captive portal detection for all OS platforms
- iptables-based traffic redirection

## 🛠️ Tech Stack

**Backend:**
- Python 3.7+
- Flask (web framework)
- Cryptography (Fernet encryption)
- RPi.GPIO (hardware control)

**System Services:**
- hostapd (access point)
- dnsmasq (DHCP/DNS)
- iptables (traffic routing)
- systemd (service management)

**Frontend:**
- Pure HTML5/CSS3 (no external dependencies)
- Vanilla JavaScript (no frameworks)
- Responsive grid layout

**Hardware:**
- Raspberry Pi Zero W / 3B+ / 4
- Tactile push button
- Optional: LED indicators

## 📸 Interface Preview

**Features:**
- Signal strength visualization with color-coded bars
- Network security indicators
- Save options for credentials
- Quick-connect for saved networks
- Real-time network refresh

## 🚀 Quick Start

### Prerequisites

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y hostapd dnsmasq iptables-persistent python3-pip

# Install Python packages
sudo pip3 install cryptography flask RPi.GPIO
```

### Installation

1. **Clone or download the project:**
```bash
sudo mkdir -p /opt/captive-portal
cd /opt/captive-portal
```

2. **Save the main script:**
Place `captive_portal.py` in `/opt/captive-portal/`

3. **Create systemd service:**
```bash
sudo nano /etc/systemd/system/captive-portal.service
```

Add the service configuration (see full setup guide in docs)

4. **Enable auto-start:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable captive-portal.service
sudo systemctl start captive-portal.service
```

### Hardware Setup (Optional)

**Button Wiring:**
- GPIO 18 (Pin 12) → One side of button
- GND (Pin 14) → Other side of button

No external resistor needed (uses internal pull-up).

## 📱 How It Works

### Boot Sequence

1. **System starts** → Captive portal service launches
2. **Checks for saved networks** → Attempts automatic connection
3. **Tests internet** → Pings multiple DNS servers
4. **Activates if needed** → Starts AP after 7 seconds without internet

### User Flow

1. **Pi loses internet** → "MyPi-Config" network appears (7 seconds delay)
2. **User connects** → Open network, no password required
3. **Browser opens** → Automatic redirect to configuration page
4. **Select network** → Choose from scanned WiFi networks
5. **Enter password** → Choose save option (encrypted/temporary/none)
6. **Connect** → Pi connects to WiFi, portal auto-closes

### Manual Activation

1. **Hold button** → Press GPIO button for 3+ seconds
2. **Portal activates** → Works even with active internet
3. **Configure** → Add new networks or switch connections
4. **Auto-shutdown** → Portal closes when done (unless manually activated)

## 🔧 Configuration

### Settings (in `captive_portal.py`)

```python
# Timing Configuration
INTERNET_CHECK_INTERVAL = 5      # Check every 5 seconds
NO_INTERNET_TIMEOUT = 7          # Activate after 7 seconds
CONNECTION_TIMEOUT = 15          # WiFi connection timeout

# Access Point
AP_SSID = "MyPi-Config"          # Change network name

# GPIO
BUTTON_PIN = 18                  # GPIO pin for button
BUTTON_HOLD_TIME = 3             # Button hold duration

# Security
CREDENTIALS_FILE = "/etc/mypi/saved_networks.json"
ENCRYPTION_KEY_FILE = "/etc/mypi/network.key"
```

### Network Customization

**Change AP Name:**
```python
AP_SSID = "YourCustomName"
```

**Add Password Protection:**
Edit `/etc/hostapd/hostapd.conf`:
```
wpa=2
wpa_passphrase=YourPassword
wpa_key_mgmt=WPA-PSK
```

## 📊 System Architecture

```
┌─────────────────────────────────────────────┐
│          Captive Portal System              │
├─────────────────────────────────────────────┤
│                                             │
│  ┌─────────────┐      ┌─────────────┐       │
│  │   Internet  │◄────►│   Monitor   │       │
│  │   Monitor   │      │   Thread    │       │
│  └─────────────┘      └─────────────┘       │
│         │                     │             │
│         ▼                     ▼             │
│  ┌─────────────┐      ┌─────────────┐       │
│  │   Network   │      │   Button    │       │
│  │   Scanner   │      │ Controller  │       │
│  └─────────────┘      └─────────────┘       │
│         │                     │             │
│         ▼                     ▼             │
│  ┌─────────────────────────────────┐        │
│  │      Flask Web Server           │        │
│  │   (Captive Portal Interface)    │        │
│  └─────────────────────────────────┘        │
│         │                                   │
│         ▼                                   │
│  ┌─────────────────────────────────┐        │
│  │   Credential Manager            │        │
│  │   (Encrypted Storage)           │        │
│  └─────────────────────────────────┘        │
│                                             │
└─────────────────────────────────────────────┘
         │                      │
         ▼                      ▼
   ┌──────────┐         ┌──────────┐
   │ hostapd  │         │ dnsmasq  │
   │   (AP)   │         │ (DHCP)   │
   └──────────┘         └──────────┘
```

## 🛡️ Security Features

- **AES-256 Encryption:** All passwords encrypted using Fernet
- **Secure File Permissions:** 0600 on credential files
- **No Plaintext Storage:** Passwords never stored unencrypted
- **Isolated Key Storage:** Encryption key separate from credentials
- **Root-only Access:** Service runs with proper privileges
- **Network Isolation:** AP clients isolated until configured

## 📝 Usage Examples

### Check Status
```bash
sudo systemctl status captive-portal.service
```

### View Live Logs
```bash
sudo journalctl -u captive-portal.service -f
```

### Manual Start/Stop
```bash
sudo systemctl start captive-portal.service
sudo systemctl stop captive-portal.service
```

### Test Button (via logs)
```bash
# Hold button for 3+ seconds, then check:
sudo journalctl -u captive-portal.service | grep -i button
```

### Check Saved Networks
```bash
sudo cat /etc/mypi/saved_networks.json
```

## 🐛 Troubleshooting

### Portal Not Starting
```bash
# Check service status
sudo systemctl status captive-portal.service

# Check for conflicting services
sudo systemctl status hostapd
sudo systemctl status NetworkManager
```

### Button Not Responding
```bash
# Check GPIO configuration
gpio readall | grep 18

# Test GPIO directly
gpio -g mode 18 up
gpio -g read 18
```

### WiFi Connection Issues
```bash
# Check interface
iwconfig wlan0

# Manual scan
sudo iwlist wlan0 scan

# Check logs
sudo journalctl -u captive-portal.service --no-pager -n 50
```

### iptables Issues
```bash
# Check NAT rules
sudo iptables -t nat -L -n -v

# Check filter rules
sudo iptables -L -n -v
```

## 🎯 Use Cases

- **Headless IoT Devices** - Configure WiFi without keyboard/monitor
- **Field Deployments** - Easy network setup for remote installations
- **Development Boards** - Quick WiFi configuration for prototypes
- **Smart Home Devices** - User-friendly network onboarding
- **Educational Projects** - Teaching networking and web development
- **Portable Devices** - Battery-powered devices with WiFi

## 🤝 Contributing

Contributions welcome! Feel free to:
- Report bugs
- Suggest features
- Submit pull requests
- Improve documentation

## 📄 License

This project is available for educational and personal use.

## 👨‍💻 Author

**Katta Sai Charan**
- GitHub: [@Charan63045](https://github.com/Charan63045)
- Email: charan89135@gmail.com
- Linkedin: www.linkedin.com/in/katta-s-930405203

## 🙏 Acknowledgments

- Raspberry Pi Foundation for excellent documentation
- Python Flask community
- Open-source networking tools (hostapd, dnsmasq)

---

⭐ **Star this repository** if you find it useful!

**Built with Python, Flask, and ❤️ for the IoT community**
