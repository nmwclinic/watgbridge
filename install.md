# Watgbridge

```bash
# Di Ubuntu VPS
apt install -y git gcc g++ make golang ffmpeg imagemagick

# Clone dan patch
git clone https://github.com/akshettrj/watgbridge.git
cd watgbridge

# Update whatsmeow
go get go.mau.fi/whatsmeow@v0.0.0-20260219150138-7ae702b1eed4
go mod tidy

# Patch context.Background()
python3 -c "
with open('whatsapp/handlers.go', 'r') as f:
    lines = f.readlines()
for i, line in enumerate(lines):
    if 'GetProfilePictureInfo(' in line and 'context.Background()' not in line:
        lines[i] = line.replace('GetProfilePictureInfo(', 'GetProfilePictureInfo(context.Background(), ')
with open('whatsapp/handlers.go', 'w') as f:
    f.writelines(lines)
"

python3 -c "
files = ['utils/telegram.go', 'utils/whatsapp.go', 'telegram/handlers.go']
replacements = [
    ('SendPresence(waTypes.', 'SendPresence(context.Background(), waTypes.'),
    ('GetGroupInfo(', 'GetGroupInfo(context.Background(), '),
    ('UpdateBlocklist(', 'UpdateBlocklist(context.Background(), '),
    ('JoinGroupWithLink(', 'JoinGroupWithLink(context.Background(), '),
    ('MarkRead(msgIds,', 'MarkRead(context.Background(), msgIds,'),
]
for fname in files:
    with open(fname, 'r') as f:
        content = f.read()
    for old, new in replacements:
        if old in content and new not in content:
            content = content.replace(old, new)
    with open(fname, 'w') as f:
        f.write(content)
print('done')
"

python3 -c "
with open('telegram/handlers.go', 'r') as f:
    lines = f.readlines()
for i, line in enumerate(lines):
    if 'GetProfilePictureInfo(' in line and 'context.Background()' not in line:
        lines[i] = line.replace('GetProfilePictureInfo(', 'GetProfilePictureInfo(context.Background(), ')
with open('telegram/handlers.go', 'w') as f:
    f.writelines(lines)
"

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
