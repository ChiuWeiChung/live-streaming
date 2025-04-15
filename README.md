# HLS 串流系統

這個專案是一個基於 NGINX-RTMP 模組的 HTTP Live Streaming (HLS) 串流系統，具有以下特點：

1. **支援 RTMP 轉 HLS**：能將 RTMP 串流自動轉換為 HLS 格式，適合網頁播放。
2. **低延遲配置**：針對 HLS 進行了低延遲優化，片段時長設為 1 秒。
3. **網頁播放器**：內建支援 HLS.js 的 HTML5 網頁播放器。
4. **跨平台兼容**：可在各種裝置上播放，包括 iOS、Android、瀏覽器等。

## 目錄
- [系統架構](#系統架構)
- [設置說明](#設置說明)
- [使用方法](#使用方法)
  - [RTMP/HLS 串流](#rtmphls-串流)
  - [TCP 直接串流](#tcp-直接串流)
  - [串流方案比較](#串流方案比較)
- [延遲優化](#延遲優化)
- [常見問題](#常見問題)

## 系統架構

1. **NGINX RTMP 伺服器**：接收 RTMP 串流並轉換為 HLS。
2. **HLS 串流服務**：通過 HTTP 提供 .m3u8 播放列表和 .ts 片段文件。
3. **網頁播放器**：使用 HLS.js 在網頁上播放串流。

## 設置說明

### 前置需求
- Docker 和 Docker Compose
- ffmpeg (用於推送串流)
- 現代網頁瀏覽器 (支援 HLS.js 或原生 HLS)

### 啟動串流伺服器
```bash
docker-compose up -d
```

這將啟動一個伺服器，開放以下端口：
- 1935: RTMP 協議 (用於接收串流)
- 8080: HTTP 服務 (用於 HLS 播放和網頁介面)

## 使用方法

本專案提供兩種主要的串流方式：RTMP/HLS 串流和 TCP 直接串流。兩種方式各有優點，可根據需求選擇。

### RTMP/HLS 串流

RTMP/HLS 串流適合需要跨平台兼容性和多客戶端連接的場景。

#### 1. 將串流推送到伺服器 (RTMP)

可以使用 ffmpeg 將視頻源推送到 RTMP 伺服器：

```bash
ffmpeg -f avfoundation -framerate 30 -video_size 1280x720 -i "0:0" \
       -c:v libx264 -preset veryfast -tune zerolatency -b:v 1500k \
       -c:a aac -b:a 128k \
       -f flv "rtmp://localhost:1935/live/test"
```

##### 參數說明
- `-f avfoundation`: macOS 的視訊捕獲框架
- `-framerate 30`: 設置捕獲幀率為每秒 30 幀
- `-video_size 1280x720`: 設置視頻分辨率為 HD (1280x720)
- `-i "0:0"`: 使用第一個視頻設備和第一個音頻設備
- `-c:v libx264 -preset veryfast -b:v 1500k`: 視頻編碼設置
- `-c:a aac -b:a 128k`: 音頻編碼設置
- `-f flv`: 輸出格式為 FLV
- `rtmp://localhost:1935/live/test`: RTMP 目標地址，其中 "test" 是串流名稱

#### 2. 觀看 HLS 串流

##### 通過網頁播放器
訪問 http://localhost:8080/ 即可在網頁上觀看串流。

##### 直接訪問 HLS 地址
HLS 播放列表地址為：http://localhost:8080/hls/test.m3u8

可以使用支援 HLS 的播放器如 VLC、mpv 等播放：
```bash
mpv http://localhost:8080/hls/test.m3u8
```

### TCP 直接串流

TCP 直接串流提供更低的延遲，適合本地網路內的點對點傳輸。

#### 1. 使用 TCP 伺服器模式串流攝像頭

在此模式下，ffmpeg 會創建一個 TCP 伺服器並等待客戶端連接：

```bash
ffmpeg -f avfoundation -framerate 30 -video_size 1280x720 -i "0:0" \
       -c:v libx264 -preset ultrafast -tune zerolatency -b:v 1000k \
       -f mpegts "tcp://127.0.0.1:9000?listen"
```

注意：在 zsh shell 中，必須使用引號包裹含有 `?` 的 URL，以避免 shell 將其解析為通配符。

#### 2. 接收和播放 TCP 串流

使用 mpv 播放器連接到 TCP 串流：

```bash
mpv tcp://127.0.0.1:9000
```

#### 3. TCP 串流參數說明

##### 伺服器端 (發送端)
- `-f avfoundation`: 使用 macOS 的多媒體框架捕獲設備
- `-framerate 30`: 設置每秒 30 幀
- `-video_size 1280x720`: HD 解析度
- `-i "0:0"`: 使用第一個攝像頭和麥克風
- `-c:v libx264`: 使用 H.264 編碼
- `-preset ultrafast`: 使用最低延遲的編碼預設
- `-tune zerolatency`: 優化低延遲設置
- `-b:v 1000k`: 設置視頻比特率
- `-f mpegts`: 使用 MPEG-TS 容器格式
- `"tcp://127.0.0.1:9000?listen"`: 創建 TCP 伺服器並監聽 9000 端口

##### 客戶端 (接收端)
- `tcp://127.0.0.1:9000`: 連接到指定的 TCP 伺服器地址和端口

### 串流方案比較

#### TCP 直接串流優點
- 更低延遲 (通常低於 1 秒)
- 設置簡單，不需要額外伺服器
- 適合本地網路內的點對點傳輸

#### RTMP/HLS 優點
- 更好的跨平台兼容性 (特別是 HLS)
- 支援多客戶端同時連接
- 可緩存和回放
- 適合通過互聯網分發

### 將 TCP 串流轉換為 RTMP/HLS

如果您希望結合兩種方式的優點，可以將 TCP 串流再轉換為 RTMP 格式，然後讓 nginx-rtmp 伺服器自動轉換為 HLS。

**重要**：在執行以下命令前，請確保已啟動 Docker 容器：
```bash
docker-compose up -d
```

然後，在另一個終端窗口執行：
```bash
# 注意：需要轉碼音頻，因為 TCP 串流中的 mp2 音頻與 FLV 容器不兼容
ffmpeg -i tcp://127.0.0.1:9000 -c:v copy -c:a aac -b:a 128k -f flv rtmp://localhost:1935/live/test
```

這樣，您就可以建立完整的低延遲串流鏈：
1. 從攝像頭捕獲視頻 → TCP 串流（低延遲點對點）
2. TCP 串流 → RTMP → HLS（網頁播放、多客戶端）

#### 參數說明
- `-i tcp://127.0.0.1:9000`: 指定 TCP 串流作為輸入源
- `-c:v copy`: 不重新編碼視頻，直接複製串流內容（保持低 CPU 使用率）
- `-c:a aac -b:a 128k`: 將音頻轉碼為 AAC 格式，這是 FLV/RTMP 支援的音頻格式
- `-f flv`: 輸出格式為 FLV（RTMP 使用的格式）
- `rtmp://localhost:1935/live/test`: RTMP 目標地址

## 延遲優化

本專案針對 HLS 進行了低延遲優化：

- HLS 片段長度設為 1 秒 (默認是 5 秒)
- HLS 播放列表長度為 20 秒 (默認是 30 秒)
- 網頁播放器配置了低延遲參數：
  - `liveSyncDuration: 2` (同步延遲 2 秒)
  - `liveMaxLatencyDuration: 4` (最大延遲 4 秒)
  - `maxBufferLength: 10` (緩衝區限制為 10 秒)

## 常見問題

### Connection refused 錯誤
如果您在連接到 RTMP 伺服器時遇到「Connection refused」錯誤，請確認：
1. Docker 容器確實在運行（可使用 `docker ps` 檢查）
2. 嘗試重啟容器：`docker-compose down` 然後 `docker-compose up -d`
3. 檢查 1935 端口是否被占用：`lsof -i :1935`

### 其他影像源的使用

#### 從 RTSP 串流轉換
```bash
ffmpeg -rtsp_transport tcp -i "rtsp://您的RTSP地址" \
       -c:v libx264 -preset veryfast -b:v 1500k \
       -c:a aac -b:a 128k \
       -f flv "rtmp://localhost:1935/live/test"
```

#### 從本地視頻檔案串流
```bash
ffmpeg -re -i "本地視頻檔案.mp4" \
       -c:v libx264 -preset veryfast -b:v 1500k \
       -c:a aac -b:a 128k \
       -f flv "rtmp://localhost:1935/live/test"
```
注意：參數 `-re` 讓 ffmpeg 以實時速度讀取輸入文件，模擬直播的效果。