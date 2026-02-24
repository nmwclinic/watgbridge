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

go build
./watgbridge
```
