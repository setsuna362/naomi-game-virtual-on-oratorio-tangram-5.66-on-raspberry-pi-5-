Build Naomi Virtual-On Oratorio Tangram 5.66 (M.S.B.S. Ver. 5.66) on Raspberry Pi 5 Step by Step
# Step 1: 完成img映像燒錄
取得Raspberry Pi Imager - https://www.raspberrypi.com/software/ <br>
取得RsplayOS系統 RePlayOS 1.6.0 - https://www.replayos.com/download/ <br>
下載檔案 replayos-base-rpi-v1.6.0.img.xz <br>
<p>
執行 Raspberry Pi Imager (v2.0.7) <br>
| <br>
選 Raspberry Pi 5 <br>
| <br>
選 使用自定義鏡像img <br>
| <br>
選以下載檔案 replayos-base-rpi-v1.6.0.img.xz <br>
| <br>
SD Card Reader USB Device(排除系統驅動器打勾) <br>
<blockquote>＊請務必確定沒有選錯寫入裝置＊ </blockquote>
| <br>
寫入 (按下我理解即可進行寫入) <br>
| <br>
寫入完成 <br></p>

# Step 2: 確認系統IP網址
將寫入完成的TF卡裝上Raspberry Pi 5，並使用靠近電源位置的Micro HDMI輸出口輸出到螢幕 <br>
在Raspberry Pi 5接上鍵盤滑鼠，之後我們會分成window側與Pi 5側兩邊說明
Pi 5側:
Raspberry Pi 5開機 <br>
| <br>
選 REPLAY OPTION (Enter) <br>
| <br>
選 INFORMATION (Enter)  <br>
| <br>
IP (確認項目) <br>
| <br>
ETHERNET IP (確認IP address) <br>
<p>
你也可以在你的電腦使用 Advanced_IP_Scanner.exe 掃描 Raspberry Pi 5 ip address</p>

# Step 3: 使用putty.exe從window側登入Pi 5側
取得putty - https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html <br>
使用putty登入剛剛的Raspberry Pi 5 ip address
<pre>
login as:root
password:replayos
</pre>
試著複製下面這行，然後在putty的畫面按滑鼠右鍵
<pre><code>ip addr show wlan0</code></pre>
是的，不用keyin，可以直接Enter就下完指令執行<br>
<br>
接著我們先直接手動修復 Wi-Fi
<pre><code>cat > /etc/wpa_supplicant/wpa_supplicant-wlan0.conf << EOF 
ctrl_interface=DIR=/var/run/wpa_supplicant
update_config=1 country=TW network={ssid="你的WiFi SSID" psk="你的WiFi密碼"} EOF</code>
</pre>
1. 殺掉所有 wpa_supplicant <br>
2. 刪掉 socket lock <br>
3. 重新啟動 wpa_supplicant（背景）<br>
<pre><code>killall wpa_supplicant
rm -f /var/run/wpa_supplicant/wlan0
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant-wlan0.conf</code></pre>
成功會看到：Successfully initialized wpa_supplicant
來確認一下WiFi是否正常工作了
<pre><code>ip addr show wlan0
</code></pre>
# Step 4: 更新系統及編碼程式，編碼flycast
<pre><code>sudo apt update</code></pre>
等系統確認那些程式可以更新，確認完後進行更新
<pre><code>sudo apt -y upgrade</code></pre>
系統更新後，我們要補充編碼程序，安裝 EGL/DRM 開發依賴，依序更新下方三條項目
<pre><code>sudo apt install -y cmake build-essential libcurl4-openssl-dev libudev-dev libxext-dev</code></pre>
<pre><code>sudo apt install -y libdrm-dev libgbm-dev libegl1-mesa-dev libgles2-mesa-dev</code></pre>
<pre><code>sudo apt install -y libasound2-dev libpulse-dev libsdl2-dev</code></pre>
到此，應該可以進行cmak，但是我們先確認是否已經安裝
<pre><code>cmake --version</code></pre>
接著我們準備要去git flycast的程式，在這之前要先設定好Pi 5系統時間，請依下列格式輸入你現在的正確日期時間
<pre><code>sudo date -s "2026-05-10 12:10:00"</code></pre>
然後就可以開始去git flycast的程式
<pre><code>git clone --recursive https://github.com/flyinghead/flycast.git</code></pre>
拿到flycast程式後先別急著編碼，我們要再去取MrShooter42的/naomi_network.cpp<br>
Fix Live Monitor in Virtual On Oratorio Tangram <br>
This fixes setting the third node to Satellite. 可以讓第三台 Pi 5以Slave加入VOOT的對戰連線進行 Live Monitor
<pre><code>wget -O /root/flycast/core/network/naomi_network.cpp \
https://raw.githubusercontent.com/MrShooter42/flycast/84485e97172bdfbcb76e544743f4850cce557ff7/core/network/naomi_network.cpp</code></pre>
接著就可以準備進行flycast正式編碼了
<pre><code>cd flycast</code></pre>
<pre><code>mkdir build && cd build</code></pre>
<pre><code>cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)</code></pre>

# Step 5: 更新系統及編碼程式，編碼flycast
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
