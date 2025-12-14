# 🔐 KryptonOS
Kali Linux Security Edition (Remastered) - Security Edition

By Kelompok 3

Panduan langkah demi langkah untuk memodifikasi Kali Linux menjadi sistem operasi khusus yang berfokus pada keamanan dengan branding khusus, penyesuaian kernel, dan alat keamanan non-standar.

# Preview
<p align="center">
<img src="https://i.ibb.co.com/1GNFc5dJ/f42c739d-5b92-489c-9321-a0f0138d620e.jpg" alt="image"></a>
</p>


<p align="center">
<img src="https://i.ibb.co.com/WW0nB16N/Virtual-Box-Krypton-OS-13-12-2025-23-42-37.png" alt="image"></a>
</p>


<p align="center">
<img src="https://i.ibb.co.com/M5nSkhCw/Virtual-Box-Krypton-OS-13-12-2025-23-42-49.png" alt="image"></a>
</p>




## 📌 0. Prerequisites
- Hardware (Minimum)
- CPU: 4 core (recommended 6)
- RAM: 6–8 GB
- Storage: ≥ 40 GB free
- Software
- Kali Linux (host system)
- Internet connection
- ISO Kali Linux Live (official)

## 📦 Install tools remaster ISO
```bash
sudo apt install -y \
  xorriso \
  squashfs-tools \
  genisoimage \
  isolinux \
  syslinux-utils \
  rsync \
  wget \
  curl \
  unzip \
  gzip \
  nano \
  tree
```

## 📥 1. Download Kali Linux ISO

Download Live ISO (not installer):

kali-linux-2025.x-live-amd64.iso

(Optional) Verify checksum:

sha256sum kali-linux-2025.x-live-amd64.iso


## 2. Prepare Working Directory
```bash
mkdir -p ~/remaster-kali
cd ~/remaster-kali
```

Directory structure:
```bash
remaster-kali/
├── iso/            # mounted original ISO
├── edit_iso/       # ISO filesystem (editable)
├── squashfs/
│   └── edit/       # extracted root filesystem
```

## 3. Mount & Copy ISO Contents
```bash
mkdir iso edit_iso
sudo mount -o loop ~/Downloads/kali-linux-*.iso iso
rsync -a iso/ edit_iso/
sudo umount iso
```

## 4. Extract filesystem.squashfs
```bash
mkdir -p squashfs
sudo unsquashfs -d squashfs/edit edit_iso/live/filesystem.squashfs
```

## 5. Enter Chroot Environment

Bind system directories:
```bash
EDIT=~/remaster-kali/squashfs/edit

sudo mount --bind /dev  $EDIT/dev
sudo mount --bind /proc $EDIT/proc
sudo mount --bind /sys  $EDIT/sys
sudo cp /etc/resolv.conf $EDIT/etc/resolv.conf

sudo chroot $EDIT /bin/bash
```

## 🎨 6. Branding & UI Customization

- 6.1 Wallpaper
```bash
mkdir -p /usr/share/backgrounds/krypton
cp krypton-wallpaper.jpg /usr/share/backgrounds/krypton/
```

- 6.2 XFCE Default Wallpaper
```bash
mkdir -p /etc/xdg/xfce4/xfconf/xfce-perchannel-xml
nano /etc/xdg/xfce4/xfconf/xfce-perchannel-xml/xfce4-desktop.xml
```
```xml
<channel name="xfce4-desktop" version="1.0">
  <property name="backdrop">
    <property name="screen0">
      <property name="monitor0">
        <property name="workspace0">
          <property name="last-image" value="/usr/share/backgrounds/krypton/krypton-wallpaper.jpg"/>
          <property name="image-style" value="5"/>
        </property>
      </property>
    </property>
  </property>
</channel>
```

## 7. Bootloader Branding

ISOLINUX splash (BIOS) :
```bash
edit_iso/isolinux/splash.png
```

- Resolution: 640×480
- 256 colors
- PNG indexed

## 8. Kernel & Security Hardening (Nilai Tambahan)

- 8.1 Kernel Boot Parameters
Edit:
```bash
edit_iso/isolinux/live.cfg
```

Tambahkan:
```bash
audit=1
```

- 8.2 Sysctl Hardening
```bash
nano /etc/sysctl.d/99-krypton.conf
```
```conf
kernel.kptr_restrict=2
kernel.randomize_va_space=2
fs.suid_dumpable=0
net.ipv4.conf.all.rp_filter=1
```

- 8.3 Kernel Modules Auto-load
```bash
nano /etc/modules-load.d/krypton.conf
```
```conf
tun
br_netfilter
overlay
```

## 9. Add Security Tools

Tools Added:

- amass – subdomain enumeration
- whatweb – web fingerprinting
- httpx – HTTP probing
- ffuf – web fuzzing

Install (inside chroot):
```bash
apt update
apt install -y amass whatweb
```

Binary tools:
```bash
mkdir -p /opt/tools
cd /opt/tools
```
```bash
wget https://github.com/projectdiscovery/httpx/releases/latest/download/httpx_linux_amd64.zip
unzip httpx_linux_amd64.zip
mv httpx /usr/local/bin/
```
```bash
wget https://github.com/ffuf/ffuf/releases/latest/download/ffuf_linux_amd64.tar.gz
tar -xzf ffuf_linux_amd64.tar.gz
mv ffuf /usr/local/bin/
```


Cleanup:
```bash
apt clean
rm -rf /var/lib/apt/lists/*
exit
```

## 🧹 10. Exit Chroot & Cleanup Mounts
```bash
sudo umount $EDIT/proc
sudo umount $EDIT/sys
sudo umount $EDIT/dev
```

## 📦 11. Rebuild filesystem.squashfs
```bash
sudo mksquashfs squashfs/edit edit_iso/live/filesystem.squashfs \
  -comp gzip -noappend -b 1M
```

## 🔁 12. Rebuild ISO
```bash
cd edit_iso
sudo find . -type f -print0 | sudo xargs -0 md5sum \
  | grep -v "./md5sum.txt" | sudo tee md5sum.txt >/dev/null

sudo xorriso -as mkisofs -iso-level 3 -r -J -l \
  -b isolinux/isolinux.bin -c isolinux/boot.cat \
  -no-emul-boot -boot-load-size 4 -boot-info-table \
  -o ~/KryptonOS.iso .
```
