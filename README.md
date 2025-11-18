# 🚀 Vpn Clients Setup on Ubuntu Debian-Based Linux Distributions (System-Wide Configuration)

# Trojan Client

## **Step 1: Download \& Prepare Trojan-Go**

1. **Install prerequisites**:
```bash
sudo apt update
sudo apt install wget unzip -y
```

2. **Download and extract Trojan-Go**:
```bash
wget https://github.com/p4gefau1t/trojan-go/releases/latest/download/trojan-go-linux-amd64.zip
unzip trojan-go-linux-amd64.zip -d trojan-go-setup
cd trojan-go-setup
chmod +x trojan-go
```


***

## **Step 2: Create Trojan-Go Config**

Inside `trojan-go-setup`:

```bash
nano config.json
```

Paste:

```json
{
  "run_type": "client",
  "local_addr": "127.0.0.1",
  "local_port": 1080,
  "remote_addr": "addYourDomain",
  "remote_port": addYourPort,
  "password": [
    "addYourPassword"
  ],
  "ssl": {
    "sni": "addYourSNI",
    "verify": false,
    "verify_hostname": false
  }
}
```

Save with **Ctrl+O**, press **Enter**, then exit with **Ctrl+X**.

***

## **Step 3: Start Trojan-Go**

```bash
./trojan-go -config ./config.json
```

If error `"bind: address already in use"`:

```bash
sudo fuser -k 1080/tcp
```
***

## **Step 4: Install \& Configure Privoxy**

1. **Install Privoxy**:
```bash
sudo apt install privoxy -y
```

2. **Edit config**:
```bash
sudo nano /etc/privoxy/config
```

At the end of the file, add:

```
forward-socks5t   /   127.0.0.1:1080 .
```

3. **Restart Privoxy**:
```bash
sudo systemctl restart privoxy
sudo systemctl enable privoxy
```

4. **Check Privoxy**:
```bash
sudo systemctl status privoxy
```

You should see **active (running)**.

***

## **Step 5: System-Wide Proxy (Environment Variables)**

```bash
sudo nano /etc/environment
```

Add at the bottom:

```bash
http_proxy="http://127.0.0.1:8118"
https_proxy="http://127.0.0.1:8118"
ftp_proxy="http://127.0.0.1:8118"
no_proxy="localhost,127.0.0.1,::1"

HTTP_PROXY="http://127.0.0.1:8118"
HTTPS_PROXY="http://127.0.0.1:8118"
FTP_PROXY="http://127.0.0.1:8118"
NO_PROXY="localhost,127.0.0.1,::1"
```

Log out \& back in, or reboot.

***

## **Step 6: Make APT Use the Proxy**

```bash
sudo nano /etc/apt/apt.conf.d/99proxy
```

Paste:

```
Acquire::http::Proxy "http://127.0.0.1:8118/";
Acquire::https::Proxy "http://127.0.0.1:8118/";
```


***

## **Step 7: Verify**

- **Check proxy variables**:

```bash
env | grep -i proxy
```

- **Test with curl/wget**:

```bash
curl -I http://example.com
wget -O - http://example.com
```

- **Test apt**:

```bash
sudo apt update
```


***

## **After Reboot**

Privoxy + system proxy configs persist automatically.
To reconnect, just restart Trojan-Go:

```bash
cd ~/trojan-go-setup
./trojan-go -config ./config.json
```


***

# VLESS Client

## **Step 1: Download and Install Xray-core**

```bash
sudo apt update
sudo apt install wget unzip -y
wget https://github.com/XTLS/Xray-core/releases/latest/download/Xray-linux-64.zip
unzip Xray-linux-64.zip -d xray
cd xray
chmod +x xray
```

## **Step 2: Create Your VLESS Config File**

```bash
nano config.json
```
Paste this configuration (based on your link):
```bash
{
  "inbounds": [
    {
      "port": 10808,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": {
        "udp": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "your-domain",
            "port": 443,
            "users": [
              {
                "id": "vless-id",
                "encryption": "none"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls",
        "tlsSettings": {
          "serverName": "your-host",
          "allowInsecure": true,
          "alpn": []
        }
      }
    }
  ]
}
```
## **Step 3: Start Xray-core**

Run:
```bash
./xray run -c config.json
```
Leave this terminal running. Your SOCKS5 proxy is now at 127.0.0.1:10808.

## **Step 4: System-wide Proxy Configuration**

### For terminal and system apps:
```bash
export http_proxy="socks5h://127.0.0.1:10808"
export https_proxy="socks5h://127.0.0.1:10808"
```
### For persistent system-wide proxy:

Edit /etc/environment and add:
```bash
http_proxy="socks5h://127.0.0.1:10808"
https_proxy="socks5h://127.0.0.1:10808"
ftp_proxy="socks5h://127.0.0.1:10808"
no_proxy="localhost,127.0.0.1,::1"
```
Reboot or log out/in.

### For package management (apt):
```bash
sudo nano /etc/apt/apt.conf.d/99proxy
```
paste:
```bash
Acquire::http::Proxy "socks5h://127.0.0.1:10808/";
Acquire::https::Proxy "socks5h://127.0.0.1:10808/";
```
## **Step 5: Browser Proxy Setup (Optional)**
- Firefox: Preferences → Network Settings → Manual → SOCKS Host 127.0.0.1, Port 10808, SOCKS v5, and enable “Proxy DNS when using SOCKS v5”.
- Chrome/Brave: start from the terminal with:
  ```bash
  google-chrome --proxy-server="socks5://127.0.0.1:10808"
  brave-browser --proxy-server="socks5://127.0.0.1:10808"
  ```
## **Step 6: (Optional) Autostart on Boot**

### **✅ OPTION A — Xray stays in your Home Folder**

**Step 1 — Create the Systemd Service File**

Step 1 — Create the Systemd Service File
```bash
sudo nano /etc/systemd/system/xray.service
```
Now paste this exactly, but replace yourusername with your real Linux username (ex: hasindu):
```bash
[Unit]
Description=Xray-core VLESS Proxy
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=yourusername
WorkingDirectory=/home/yourusername/xray
ExecStart=/home/yourusername/xray/xray run -c /home/yourusername/xray/config.json
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Save & Exit:
- Press CTRL + O → Enter
- Press CTRL + X

**Step 2 — Apply the Service & Enable Auto-Start**
```bash
sudo systemctl daemon-reload
sudo systemctl enable xray.service
sudo systemctl start xray.service
```
**Step 3 — Check Status**
```bash
sudo systemctl status xray.service
```
You should see active (running) in green. Also confirm SOCKS proxy port:
```bash
ss -tlnp | grep 10808
```
If you see it listening → success ✅

### **✅ OPTION B — Recommended (Move Xray to System Path)**

**Step 1 — Move Xray to /opt/xray**
```bash
sudo mkdir -p /opt/xray
sudo cp -a ~/xray/. /opt/xray/
sudo chown -R root:root /opt/xray
```
**Step 2 — Create/Modify Service File**
```bash
sudo nano /etc/systemd/system/xray.service
```
```bash
[Unit]
Description=Xray-core VLESS Proxy
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/xray
ExecStart=/opt/xray/xray run -c /opt/xray/config.json
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```
Save and exit

**Step 3 — Enable Auto-Start**
```bash
sudo systemctl daemon-reload
sudo systemctl enable xray.service
sudo systemctl start xray.service
```

**Step 4 — View Live Logs**
```bash
journalctl -u xray.service -f
```
This helps you see connection errors in real-time.





# 👉 Happy...

