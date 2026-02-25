# Watgbridge

```bash
# Di Ubuntu VPS
apt install -y git gcc g++ make golang ffmpeg imagemagick

go mod tidy
go build

./watgbridge
```

## Install Services

```bash
# Cek path binary dulu
pwd

# Lalu buat service file:
sudo nano /etc/systemd/system/watgbridge.service
```

```ini
[Unit]
Description=WaTgBridge - WhatsApp Telegram Bridge
After=network.target

[Service]
Type=simple
User=harisinside
WorkingDirectory=/vdb1/sites/watgbridge
ExecStart=/vdb1/sites/watgbridge/watgbridge
Restart=on-failure
RestartSec=5s

# Restart setiap 24 jam (rekomendasi developer)
RuntimeMaxSec=86400

[Install]
WantedBy=multi-user.target
```

```bash
# Aktifkan
sudo systemctl daemon-reload
sudo systemctl enable watgbridge
sudo systemctl start watgbridge
sudo systemctl status watgbridge

journalctl -u watgbridge -f

# RuntimeMaxSec=86400 itu yang bikin auto restart tiap 24 jam sesuai rekomendasi developer karena WhatsApp sering disconnect.
```
