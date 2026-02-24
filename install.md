```bash
# 1. Install dependencies
apt install -y git gcc golang ffmpeg imagemagick

# 2. Clone repo
cd /vdb1/server
git clone https://github.com/akshettrj/watgbridge.git
cd watgbridge

# 3. Update whatsmeow ke versi terbaru
nano go.mod
# ubah baris whatsmeow ke:
# go.mau.fi/whatsmeow v0.0.0-20260219150138-7ae702b1eed4

rm go.sum
go mod tidy

# 4. Build
go build

# 5. Setup config
cp sample_config.yaml config.yaml
nano config.yaml

# 6. Jalankan pertama kali untuk scan QR
./watgbridge
```

Setelah QR ter-scan dan login sukses, baru setup systemd:

```bash
# Copy sample service
cp watgbridge.service.sample /etc/systemd/system/watgbridge.service
nano /etc/systemd/system/watgbridge.service
# Edit User dan ExecStart sesuai path

systemctl daemon-reload
systemctl enable watgbridge
systemctl start watgbridge
systemctl status watgbridge
```