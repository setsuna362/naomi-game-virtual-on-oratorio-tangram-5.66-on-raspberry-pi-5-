# Naomi-Game-Virtual-On-Oratorio-Tangram-5.66-on-Raspberry-Pi-5-
Build Naomi Virtual-On Oratorio Tangram 5.66 (M.S.B.S. Ver. 5.66) on Raspberry Pi 5 Step by Step

Raspberry Pi Imager (v2.0.7)
|
Raspberry Pi 5
|
使用自定義鏡像img
|
replayos-base-rpi-v1.6.0.img.xz
|
SD Card Reader USB Device(排除系統驅動器打勾)
|
寫入 (我理解)
|
寫入完成

==================================================
開機
|
REPLAY OPTION
|
INFORMATION
|
IP
|
ETHERNET IP

==================================================

login as:root
password:replayos
==================================================



OPTION:

ip addr show wlan0

先直接手動修復 Wi-Fi

cat > /etc/wpa_supplicant/wpa_supplicant-wlan0.conf <<EOF
ctrl_interface=DIR=/var/run/wpa_supplicant
update_config=1
country=TW

network={
    ssid="你的WiFi"
    psk="你的密碼"
}
EOF

如果沒有 IP：

先把舊的 wpa_supplicant 清掉。

1. 殺掉所有 wpa_supplicant
killall wpa_supplicant
2. 刪掉 socket lock
rm -f /var/run/wpa_supplicant/wlan0

重新啟動 wpa_supplicant（背景）
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant-wlan0.conf

成功會看到：
Successfully initialized wpa_supplicant

ip addr show wlan0

======================================
$ sudo apt update

$ sudo apt -y upgrade

$ sudo apt install -y cmake build-essential libcurl4-openssl-dev libudev-dev libxext-dev

🚀 Step 2：安裝 EGL/DRM 開發依賴

先補齊 Pi 的 graphics stack：

$ sudo apt install -y libdrm-dev libgbm-dev libegl1-mesa-dev libgles2-mesa-dev

# $ sudo apt install -y libasound2-dev libpulse-dev libsdl2-dev

$ cmake --version
確認是否已經安裝

$ sudo date -s "2026-05-10 12:10:00"

$ git clone --recursive https://github.com/flyinghead/flycast.git

$ wget -O /root/flycast/core/network/naomi_network.cpp \
https://raw.githubusercontent.com/MrShooter42/flycast/84485e97172bdfbcb76e544743f4850cce557ff7/core/network/naomi_network.cpp

$ cd flycast

$ mkdir build && cd build


cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
==========================================
use WinSCP or FileZilla put

naomi.zip
vonot.zip

in /root/flycast/build


==============================================

1. 停用 replay frontend
systemctl disable replay.service
systemctl stop replay.service

find ~/flycast/ -type f -exec chmod 755 {} \; && find ~/flycast/ -type d -exec chmod 755 {} \;

exec /root/flycast/build/flycast "$@"

到遊戲端按一下setting
左上角按一下 Done

回到電腦端按 Ctrl+C
自動退出 putty.exe

此時系統emu.cfg建立完成



==============================================
use WinSCP or FileZilla put

Ctrl+O (大寫英文字O)
open directory:

/root/.config/flycast

click "OK"
put

SDL_Generic X-Box pad_HDR-0040.cfg
SDL_Generic X-Box pad_VIRTUAL-ON ORATORIO TANGRAM_arcade.cfg

SDL_HORI TWIN STICK 3_HDR-0040.cfg
SDL_HORI TWIN STICK 3_VIRTUAL-ON ORATORIO TANGRAM_arcade.cfg

/root/.config/flycast/mappings/


==============================================

重新由putty.exe登入

login as:root
password:replayos


nano /root/.config/flycast/emu.cfg

emu.cfg (DNA ActAsServer = yes, RNA ActAsServer = no)

[HDR-0040]
input.device1 = 8
input.device1.1 = 10
input.device1.2 = 10
input.device2 = 0
input.device2.1 = 1
input.device2.2 = 1
config.PerGameVmu = no

[VIRTUAL-ON ORATORIO TANGRAM]
config.PerGameVmu = no
config.ForceFreePlay = no
input.device1 = 8
input.device1.1 = 10
input.device1.UseNetworkExpansionDevices = yes
network.ActAsServer = no
network.DCNet = no
network.Enable = no
network.EnableUPnP = no


[config]
Dreamcast.ContentPath = /media/sd/roms/sega_dc
pvr.rend = 4
rend.Resolution = 1080

[window]
fullscreen = no
height =1080
width = 1920
===========
Ctrl+O
Enter
Ctrl+X
===========================================

nano /etc/systemd/system/flycast.service

改成這樣👇（很重要）
===========
[Unit]
Description=Flycast Kiosk
After=systemd-user-sessions.service getty@tty1.service
Requires=systemd-user-sessions.service

[Service]
User=root
WorkingDirectory=/root/flycast/build

Environment=SDL_VIDEODRIVER=kmsdrm
Environment=SDL_AUDIODRIVER=alsa

ExecStart=/root/flycast/build/flycast /root/flycast/build/vonot.zip

Restart=always
RestartSec=2

StandardInput=tty
TTYPath=/dev/tty1
TTYReset=yes
TTYVHangup=yes

[Install]
WantedBy=graphical.target
===========
Ctrl+O
Enter
Ctrl+X
===========
4. Flycast 設為開機服務 reload + enable

systemctl daemon-reload
systemctl enable flycast.service

5.reboot
reboot




==========================================

設定固定 Wifi IP

ip addr

inet 192.168.68.55/22

編輯：

nano /etc/dhcpcd.conf

最下面加入：


interface wlan0
static ip_address=192.168.68.250/22
static routers=192.168.68.1
static domain_name_servers=8.8.8.8 1.1.1.1

reboot

===============================================
如何停止


systemctl disable flycast.service
systemctl stop flycast.service

先確認 service 狀態（重要）

請先看：

systemctl status flycast.service

大概率你會看到：

○ flycast.service - Flycast Kiosk
     Loaded: loaded (/etc/systemd/system/flycast.service; disabled; preset: enabled)
     Active: inactive (dead)

修改

nano /root/.config/flycast/emu.cfg

systemctl daemon-reload
systemctl enable flycast.service
systemctl start flycast.service

然後：

systemctl status flycast.service


===============================================
cd /root/flycast/build
./flycast "/media/sd/roms/sega_dc/Dennou Senki - Virtual-On - Oratorio Tangram (Japan)/Dennou Senki - Virtual-On - Oratorio Tangram (Japan).gdi"
exec /root/flycast/build/flycast "$@"
/root/.config/flycast
/root/.local/share/flycast

How to Utilize VMU in Flycast (2026):

Flycast Settings: Navigate to controller settings in the emulator.

Enable VMS: Enable "Use Physical VMU Memory" and "Use Network Expansion Devices".

Load Data: Use a NAOMI-compatible save file containing your customized VOOT data.

These improvements make it possible to use customized virtuaroids (color schemes/emblems)
 and custom quick messages, which were previously limited to arcade setups with the special 
Dreamcast Customization Disc.
