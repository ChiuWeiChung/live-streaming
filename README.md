# UDP 網路攝像頭串流系統

這個專案是一個低延遲 UDP 網路攝像頭串流系統，主要提供以下功能：

1. **即時攝像頭串流**：使用 ffmpeg 捕獲本地網路攝像頭影像，通過 UDP 協議進行傳輸。

2. **極低延遲播放**：使用多種低延遲優化技術，確保視頻傳輸和播放的時延最小化。

3. **本地網路傳輸**：適用於同一網路內的設備間視頻傳輸，無需透過伺服器中轉。

整個系統非常適合需要實時視頻傳輸的場景，例如視頻監控、即時互動等應用。

## 使用說明

### 1. 列出本地的網路攝像頭

在開始串流前，可以使用以下命令列出系統可用的網路攝像頭：

```bash
ffmpeg -f avfoundation -list_devices true -i ""
```

### 2. 串流與接收

#### 超低延遲設定（推薦）

**在接收端監聽串流：**

```bash
mpv --no-cache --profile=low-latency --video-sync=audio \
    --demuxer-max-bytes=256k \
    udp://localhost:6666
```

**在發送端推送串流：**

```bash
ffmpeg \
  -f avfoundation \
  -framerate 30 \
  -video_size 1280x720 \
  -i "0:0" \
  \
  -b:v 1000k \
  -c:v libx264 \
  -preset ultrafast \
  -tune zerolatency \
  -g 30 \
  \
  -fflags nobuffer \
  -flags low_delay \
  -flush_packets 1 \
  -max_delay 0 \
  \
  -f mpegts \
  udp://localhost:6666  
```

#### 較高畫質設定（延遲較大）

**在接收端監聽串流：**

```bash
mpv --no-cache --video-sync=audio \
    udp://localhost:6666
```

**在發送端推送串流：**

```bash
ffmpeg \
  -f avfoundation \
  -framerate 30 \
  -video_size 1280x720 \
  -i "0:0" \
  -c:v libx264 \
  -preset medium \
  -tune zerolatency \
  -b:v 20M \
  -maxrate 20M \
  -bufsize 10M \
  -max_delay 0 \
  -f mpegts \
  udp://localhost:6666    
```

## 參數說明

### 監聽串流的參數

* **udp://localhost:6666**:
	* 指定要監聽的 UDP 流地址（來自 localhost 的 6666 端口）。
* **--no-cache**:
	* 禁用緩衝區，直接播放數據，減少延遲。
* **--profile=low-latency**:
	* 使用低延遲配置，以提高實時播放性能。
* **--video-sync=audio**:
	* 將視頻同步到音頻，減少音視頻不同步的問題。
* **--demuxer-max-bytes=256k**:
	* 設置最大緩衝大小為 256KB，適合低延遲播放。

### 輸出串流的參數

#### 輸入相關參數
* **-f avfoundation**:
	* 使用 macOS 的多媒體框架捕捉攝像頭和麥克風。
* **-framerate 30**:
	* 設置捕捉幀率為每秒 30 幀。
* **-video_size 1280x720**:
	* 設置捕捉視頻的分辨率為 1280x720（HD）。
* **-i "0:0"**:
	* 指定輸入設備：
	* 第一個視頻設備（如默認攝像頭）。

#### 視頻編碼相關參數
* **-b:v 1000k**:
	* 設置視頻比特率為 1000 kbps，適合低延遲流。
* **-c:v libx264**:
	* 指定視頻編碼器為 H.264。
* **-preset ultrafast**:
	* 使用 H.264 編碼器的最快預設，減少編碼延遲。
* **-tune zerolatency**:
	* 調整編碼器以優化低延遲直播。
* **-g 30**:
	* 設置 GOP（幀組）的大小為 30，意味著每 30 幀插入一個關鍵幀。

#### 延遲優化參數
* **-fflags nobuffer**:
	* 禁用內部緩衝區，減少延遲。
* **-flags low_delay**:
	* 啟用低延遲編碼模式。
* **-flush_packets 1**:
	* 強制在每個數據包後刷新輸出緩衝。
* **-max_delay 0**:
	* 將最大延遲設置為零，確保最小延遲。

#### 編碼器和輸出格式
* **-f mpegts**:
	* 指定輸出格式為 MPEG-TS（傳輸流格式）。
* **udp://localhost:6666**:
	* 將流推送到本地的 UDP 端口 6666。

## 高品質參數說明

* **-preset medium**:
	* 比 ultrafast 提供更好的壓縮效率，但會增加一些延遲。
* **-b:v 20M**:
	* 設置更高的比特率（20 Mbps）以提高畫質。
* **-maxrate 20M**:
	* 設置最大比特率為 20 Mbps。
* **-bufsize 10M**:
	* 設置編碼緩衝區大小為 10 MB，允許更好的品質控制。