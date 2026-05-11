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

# Step 2: 確認系統 IP 位址
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
<pre>login as:root
password:replayos</pre>
試著複製下面這行，然後在putty的畫面按滑鼠右鍵
<pre><code>ip addr show wlan0</code></pre>
是的，不用 keyin，可以直接 Enter 就下完指令執行<br>
<p>而且你可以按 ↑ 或 ↓ 去選擇曾經在 putty 內執行過的指令直接執行</p>
<p>接下來只要是可以複製的指令，我們都在 putty 內進行</p>
<p>意義上就是遠端控制 Pi 5 進行安裝指令</p>
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
<P></P>MrShooter42 Fix Live Monitor in Virtual On Oratorio Tangram. This fixes setting the third node to Satellite.</P> 
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
<h4>window 側: </h4>
停用 ReplayOS原本運作的介面 replay frontend
<pre><code></code>systemctl disable replay.service</code></pre>
<pre><code></code>systemctl stop replay.service</code></pre>
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
<h4>window 側: </h4>
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
用 nano 的語法打開 emu.cfg 後，新增以下 👇 的內容到 emu.cfg 內最上方<br>
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
<P></P>確認都沒有問題後，按下 Ctrl + C 中斷遊戲運行</P>
<p>👇</P>
<P>接下來emu.cfg的設定會有三個分歧點: 單機、2台連線、衛星Satellite Monitor Live </p>
1. 單機 Pi 5 進行遊戲，要用這個設定 
<pre>network.ActAsServer = no
network.Enable = no</pre>
2. 2台 Pi 5 進行連線遊戲，必須有一台是 Master 主機，一台是 Slave 主機<br>
要修改 emu.cfg 內[VIRTUAL-ON ORATORIO TANGRAM]這一段的設定
Master 主機使用設定：<pre>network.ActAsServer = yes
network.Enable = yes</pre>
Slave 主機使用設定：<pre>network.ActAsServer = no
network.Enable = yes</pre>
衛星Satellite Monitor Live 主機使用設定與 Slave 主機相同：<pre>network.ActAsServer = no
network.Enable = yes</pre>
取決於連線 Master 主機的順序

# Step 8: Pi 5 系統設定自動開機運行
<h4>window 側: </h4>
使用 putty 登入 Raspberry Pi 5
<pre>login as:root
password:replayos</pre>
建立 service 設定檔
<pre><code>nano /etc/systemd/system/flycast.service</code></pre>
貼上下方的 code 👇
<pre><code>
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
WantedBy=graphical.target</code></pre>
改完所有參數之後，按 Ctrl+O 進行存檔<br>
Enter 同意覆寫<br>
Ctrl+X 離開<br>

Flycast 設為開機服務 reload + enable

<pre><code>systemctl daemon-reload</code></pre>
<pre><code>systemctl enable flycast.service</code></pre>

<pre><code>reboot</code></pre>
至此，教學完畢

# Step EX 1: 已經自動開機運行，之後我要修改 emu.cfg 應該要怎麼做
<h4>window 側: </h4>
<p>由於已經啟動持續自動運行，在 Linux 的系統上，程序關閉時會自動存檔現在運行中的 emu.cfg</p>
<p>所以你使用 FTP丟入系統覆蓋 emu.cfg 是沒有用的，現有的程序停止時會將已上傳的 emu.cfg 覆蓋掉</p>
<p>因次我們必須先停止 Pi 5 自動開機，才能進行 emu.cfg 修改</p>
使用 putty 登入 Raspberry Pi 5
<pre>login as:root
password:replayos</pre>
停止程序
<pre><code>systemctl disable flycast.service</code></pre>
<pre><code>systemctl stop flycast.service</code></pre>

先確認 service 狀態（重要）
<pre><code>systemctl status flycast.service</code></pre>

你會看到
<pre>flycast.service - Flycast Kiosk
Loaded: loaded (/etc/systemd/system/flycast.service; disabled; preset: enabled)
Active: inactive (dead) </pre>
執行修改
<pre><code>nano /root/.config/flycast/emu.cfg</code></pre>
<p>接著修改你想要變更的部分</P>
<p>修改完成後，你可以執行確認效果是否是你想要的</P>

<pre><code>cd flycast</code></pre>
<pre><code>cd build</code></pre>
<pre><code>./flycast vonot.zip</code></pre>

確認無誤之後恢復自動開機運行
<pre><code>systemctl daemon-reload</code></pre>
<pre><code>systemctl enable flycast.service</code></pre>
<pre><code>systemctl start flycast.service</code></pre>

然後確認 service 狀態（重要）
<pre><code>systemctl status flycast.service</code></pre>

# Step EX 2: 我要執行 Dreamcast 版本的 Virtual-On Oratorio Tangram 5.45 要怎麼做
<h4>window 側: </h4>
<p>我們必須先停止 Pi 5 自動開機，才能進行手動運行 flycast </p>
使用 putty 登入 Raspberry Pi 5
<pre>login as:root
password:replayos</pre>
停止程序
<pre><code>systemctl disable flycast.service</code></pre>
<pre><code>systemctl stop flycast.service</code></pre>
確認 service 狀態（重要）
<pre><code>systemctl status flycast.service</code></pre>

<p>請使用 WinSCP 或 FileZilla 這類的FTP軟體</p>
<p>將 Dennou Senki - Virtual-On - Oratorio Tangram (Japan) 目錄 copy 至 <pre>/media/sd/roms/sega_dc/</pre>
Dennou Senki - Virtual-On - Oratorio Tangram (Japan) 目錄內需要有 <br>
Dennou Senki - Virtual-On - Oratorio Tangram (Japan).gdi <br>
及 Dennou Senki - Virtual-On - Oratorio Tangram (Japan) (Track 01).bin ~ Dennou Senki - Virtual-On - Oratorio Tangram (Japan) (Track 58).bin <br>
共 59 個檔案</p>
FTP 連線到 Pi 5 的 ip address 帳號 root 密碼 replayos<br>

使用 putty
<pre><code>cd /root/flycast/build</code></pre>
<pre><code>./flycast "/media/sd/roms/sega_dc/Dennou Senki - Virtual-On - Oratorio Tangram (Japan)/Dennou Senki - Virtual-On - Oratorio Tangram (Japan).gdi"</code></pre>
遊戲結束後，回到電腦端 putty 這邊，按 Ctrl+C 中斷 flycast 程式<br>
視你的需求進入 Naomi 的 Virtual-On - Oratorio Tangram 進行確認 (看 VMU 是否有自己修改好的機體)<br>
VMU 存檔位於
<pre>/root/.local/share/flycast</pre>pre>

確認無誤之後視你的需求恢復自動開機運行
<pre><code>systemctl daemon-reload</code></pre>
<pre><code>systemctl enable flycast.service</code></pre>
<pre><code>systemctl start flycast.service</code></pre>

然後確認 service 狀態（重要）
<pre><code>systemctl status flycast.service</code></pre>

# Step EX 3: 我想要執行 Dreamcast 版本的 Phantasy Star Online Ver. 2 要怎麼做
<h4>window 側: </h4>
使用 putty 登入 Raspberry Pi 5
<pre>login as:root
password:replayos</pre>
停止程序
<pre><code>systemctl disable flycast.service</code></pre>
<pre><code>systemctl stop flycast.service</code></pre>
確認 service 狀態（重要）
<pre><code>systemctl status flycast.service</code></pre>

<p>請使用 WinSCP 或 FileZilla 這類的FTP軟體</p>
<p>將 Phantasy Star Online Ver. 2 (USA) (En,Ja,Fr,De,Es) (IVES Patched) 目錄 copy 至 <pre>/media/sd/roms/sega_dc/</pre>
Phantasy Star Online Ver. 2 (USA) (En,Ja,Fr,De,Es) (IVES Patched) 目錄內需要有 <br>
Phantasy Star Online Ver. 2 EP (USA) (En,Ja,Fr,De,Es) (IVES Patched).cdi <br>
共 1 個檔案</p>
FTP 連線到 Pi 5 的 ip address 帳號 root 密碼 replayos<br>
<pre><code>exec /root/flycast/build/flycast "$@"</code></pre>
<h4>Pi 5 側: </h4>
Play!
