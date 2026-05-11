Build Naomi Virtual-On Oratorio Tangram 5.66 (M.S.B.S. Ver. 5.66) on Raspberry Pi 5 Step by Step
# Step 1: 完成img映像燒錄
<h4>window 側: </h4>
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
在Raspberry Pi 5 接上鍵盤滑鼠，之後我們會分成 window側 與 Pi 5側 兩邊說明 <br>
<h4>Pi 5側: </h4>
Raspberry Pi 5開機 <br>
| <br>
選 REPLAY OPTION (Enter) <br>
| <br>
選 INFORMATION (Enter)  <br>
| <br>
IP (確認項目) <br>
| <br>
ETHERNET IP (確認IP address) <br>
<p>你也可以在你的電腦使用 Advanced_IP_Scanner.exe 掃描 Raspberry Pi 5 ip address</p>

# Step 3: 使用 putty.exe 從 window 側登入 Pi 5側
<h4>window 側: </h4>
取得 putty - https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html <br>
使用 putty 登入剛剛的 Raspberry Pi 5 ip address
<pre>
login as:root
password:replayos
</pre>
試著複製下面這行，然後在putty的畫面按滑鼠右鍵
<pre><code>ip addr show wlan0</code></pre>
是的，不用 keyin，可以直接 Enter 就下完指令執行<br>
<p>而且你可以按 ↑ 或 ↓ 去選擇曾經在 putty 內執行過的指令直接執行</p>
接下來只要是可以複製的指令，我們都在 putty 內進行<br>
意義上就是遠端控制 Pi 5 進行安裝指令<br>
首先我們先直接手動修復 WiFi
<pre><code>cat > /etc/wpa_supplicant/wpa_supplicant-wlan0.conf << EOF 
ctrl_interface=DIR=/var/run/wpa_supplicant
update_config=1 country=TW network={ssid="你的WiFi SSID" psk="你的WiFi密碼"} EOF</code></pre>
1. 殺掉所有 wpa_supplicant <br>
2. 刪掉 socket lock <br>
3. 重新啟動 wpa_supplicant（背景）<br>
<pre><code>killall wpa_supplicant
rm -f /var/run/wpa_supplicant/wlan0
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant-wlan0.conf</code></pre>
成功會看到：Successfully initialized wpa_supplicant
來確認一下 WiFi 是否正常工作了
<pre><code>ip addr show wlan0</code></pre>
# Step 4: 更新系統及編碼程式，編碼flycast
<h4>window 側: </h4>
<pre><code>sudo apt update</code></pre>
這個指令是讓系統確認那些程式需要更新，待確認完後我們開始進行更新
<pre><code>sudo apt -y upgrade</code></pre>
系統更新完成後，我們要補充編碼程序，安裝 EGL/DRM 開發依賴<br>
依序執行下方三條項目進行更新
<pre><code>sudo apt install -y cmake build-essential libcurl4-openssl-dev libudev-dev libxext-dev</code></pre>
<pre><code>sudo apt install -y libdrm-dev libgbm-dev libegl1-mesa-dev libgles2-mesa-dev</code></pre>
<pre><code>sudo apt install -y libasound2-dev libpulse-dev libsdl2-dev</code></pre>
到此，應該可以進行 cmake，但是我們先確認是否已經安裝
<pre><code>cmake --version</code></pre>
接著我們準備要去 git flycast 的程式，在這之前要先設定好Pi 5系統時間，請依下列格式輸入你現在的正確日期時間並執行
<pre><code>sudo date -s "2026-05-10 12:10:00"</code></pre>
接下來開始 git flycast 的程式
<pre><code>git clone --recursive https://github.com/flyinghead/flycast.git</code></pre>
拿到 flycast 程式後先別急著編碼，我們要再去取 MrShooter42 的 naomi_network.cpp<br>
MrShooter42 Fix Live Monitor in Virtual On Oratorio Tangram. This fixes setting the third node to Satellite.<br> 
用這個 network cpp 編碼進 flycast，可以讓第三台 Pi 5以Slave 主機的方式加入 VOOT 的對戰連線進行 Live Monitor
<pre><code>wget -O /root/flycast/core/network/naomi_network.cpp \
https://raw.githubusercontent.com/MrShooter42/flycast/84485e97172bdfbcb76e544743f4850cce557ff7/core/network/naomi_network.cpp</code></pre>
接著就可以準備進行flycast正式編碼了
<pre><code>cd flycast</code></pre>
<pre><code>mkdir build && cd build</code></pre>
<pre><code>cmake .. -DCMAKE_BUILD_TYPE=Release</code></pre>
<pre><code>make -j$(nproc)</code></pre>
# Step 5: 將NAOMI Rom及bios放到 Pi 5 內
<h4>window 側: </h4>
請使用 WinSCP 或 FileZilla 這類的FTP軟體
將這兩個檔案
<pre>naomi.zip
vonot.zip</pre>
放到 Pi 5 <pre>/root/flycast/build</pre>的目錄下，Virtual-On-Oratorio-Tangram-5.66的運行只需要這兩個檔案<br>
FTP 連線到 Pi 5 的 ip address 帳號 root 密碼 replayos<br>
到這 Step 有問題的，先去學習一下 FTP 軟體如何使用，你可以自行尋找 Youtube 教學影片觀看
# Step 6: 停止ReplayOS的UI介面，完成 flycast 系統設定檔 emu.cfg 建立
停用 replay frontend
<pre><code></code>systemctl disable replay.service
systemctl stop replay.service</code></pre>
此時你會看到 Pi 5 輸出的 HDMI 螢幕畫面變黑，表示 ReplayOS 已經永久被我們關閉<br>
接著我們執行下列程式設定 Pi 5 內 flycast 的資料存取運行權限
<pre><code>find ~/flycast/ -type f -exec chmod 755 {} \; && find ~/flycast/ -type d -exec chmod 755 {} \;</code></pre>
接著我們第一次 flycast 啟動
<pre><code>exec /root/flycast/build/flycast "$@"</code></pre>
這時你會看到 Pi 5 輸出的 HDMI 螢幕變成我們熟悉的 flycast 操作介面<br>
<h4>Pi 5 側: </h4>
我們控制 Pi 5 測的滑鼠，到遊戲端按一下setting <br>
然後按一下左上角的 Done <br></p>
<h4>window 側: </h4>
回到電腦端 putty 這邊，按 Ctrl+C 中斷 flycast 程式<br>
中斷後會自動退出 putty.exe <br>
此時 Pi 5 系統內 flycast 的 emu.cfg 建立完成

# Step 7: 修改 flycast 統設定檔 emu.cfg
請使用 WinSCP 或 FileZilla 這類的FTP軟體，連接上 Pi 5 <br>
將網頁上的4個 .cfg檔
<pre>
SDL_Generic X-Box pad_HDR-0040.cfg
SDL_Generic X-Box pad_VIRTUAL-ON ORATORIO TANGRAM_arcade.cfg
SDL_HORI TWIN STICK 3_HDR-0040.cfg
SDL_HORI TWIN STICK 3_VIRTUAL-ON ORATORIO TANGRAM_arcade.cfg</pre>
放到 /root/.config/flycast/mappings/ 目錄下<br>
重新由 putty.exe 登入
<pre>login as:root
password:replayos</pre>
進行 emu.cfg 編寫
<pre><code>nano /root/.config/flycast/emu.cfg</code></pre>
用 nano 的語法打開 emu.cfg 後，新增以下的內容到 emu.cfg 內最上方<br>
<pre><code>[HDR-0040]
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
network.EnableUPnP = no</code></pre>
貼完之後接著找到下方的參數進行數據修改
<pre>[config]
Dreamcast.ContentPath = /media/sd/roms/sega_dc
pvr.rend = 4
rend.Resolution = 1080

[window]
fullscreen = no
height =1080
width = 1920</pre>
改完所有參數之後，按 Ctrl+O 進行存檔<br>
Enter 同意覆寫<br>
Ctrl+X 離開<br>
<p>我們來測試一下修改好 emu.cfg 的 flycast 執行效果</p>
<pre><code>cd flycast</code></pre>
<pre><code>cd build</code></pre>
<pre><code>./flycast vonot.zip</code></pre>
看一下 Pi 5 輸出的 HDMI 螢幕變成我們熟悉的 Virtual-On Oratorio Tangram 5.66 (M.S.B.S. Ver. 5.66) <br>
確認都沒有問題後，按下 Ctrl + C 中斷遊戲運行

接下來會有分歧點，因為 2台 Pi 5 連線，必須有一台是 Master 主機，一台是 Slave 主機
主要的 Master 主機(假設為DNA)，emu.cfg 須設定 ActAsServer = yes <br>
連接的 Slave 主機(假設為RNA)，emu.cfg 須設定 ActAsServer = no <br>

# Step 8:  flycast 統設定檔 emu.cfg
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



如何停止

systemctl disable flycast.service
systemctl stop flycast.service

先確認 service 狀態（重要）

請先看：

systemctl status flycast.service

你會看到：

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

cd /root/flycast/build
./flycast "/media/sd/roms/sega_dc/Dennou Senki - Virtual-On - Oratorio Tangram (Japan)/Dennou Senki - Virtual-On - Oratorio Tangram (Japan).gdi"

exec /root/flycast/build/flycast "$@"

/root/.config/flycast
/root/.local/share/flycast

