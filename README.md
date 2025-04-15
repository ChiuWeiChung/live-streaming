# HLS 串流系統

這個專案是一個基於 NGINX-RTMP 模組的 HTTP Live Streaming (HLS) 串流系統，具有以下特點：

1. **支援 RTMP 轉 HLS**：能將 RTMP 串流自動轉換為 HLS 格式，適合網頁播放。
2. **低延遲配置**：針對 HLS 進行了低延遲優化，片段時長設為 1 秒。
3. **網頁播放器**：內建支援 HLS.js 的 HTML5 網頁播放器。
4. **跨平台兼容**：可在各種裝置上播放，包括 iOS、Android、瀏覽器等。

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

## 使用說明

### 1. 將串流推送到伺服器 (RTMP)

可以使用 ffmpeg 將視頻源推送到 RTMP 伺服器：

<!-- ```bash
ffmpeg -rtsp_transport tcp -i "你的影像源位置" \
       -c:v libx264 -preset veryfast -b:v 1500k \
       -c:a aac -b:a 128k \
       -f flv "rtmp://localhost:1935/live/test"
``` -->

```bash
ffmpeg -f avfoundation -framerate 30 -video_size 1280x720 -i "0:0" \
       -c:v libx264 -preset veryfast -tune zerolatency -b:v 1500k \
       -c:a aac -b:a 128k \
       -f flv "rtmp://localhost:1935/live/test"
```

#### 參數說明
- `-rtsp_transport tcp`: 如果源是 RTSP，使用 TCP 傳輸
- `-i "影像源位置"`: 指定輸入源 (可以是攝像頭、RTSP 流等)
- `-c:v libx264 -preset veryfast -b:v 1500k`: 視頻編碼設置
- `-c:a aac -b:a 128k`: 音頻編碼設置
- `-f flv`: 輸出格式為 FLV
- `rtmp://localhost:1935/live/test`: RTMP 目標地址，其中 "test" 是串流名稱

### 2. 觀看 HLS 串流

#### 通過網頁播放器
訪問 http://localhost:8080/ 即可在網頁上觀看串流。

#### 直接訪問 HLS 地址
HLS 播放列表地址為：http://localhost:8080/hls/test.m3u8

可以使用支援 HLS 的播放器如 VLC、mpv 等播放：
```bash
mpv http://localhost:8080/hls/test.m3u8
```

## 延遲優化

本專案針對 HLS 進行了低延遲優化：

- HLS 片段長度設為 1 秒 (默認是 5 秒)
- HLS 播放列表長度為 20 秒 (默認是 30 秒)
- 網頁播放器配置了低延遲參數：
  - `liveSyncDuration: 2` (同步延遲 2 秒)
  - `liveMaxLatencyDuration: 4` (最大延遲 4 秒)
  - `maxBufferLength: 10` (緩衝區限制為 10 秒)

## 其他範例

### 從其他 RTMP 串流轉換
```bash
ffmpeg -rtsp_transport tcp -i "rtsp://807e9439d5ca.entrypoint.cloud.wowza.com:1935/app-rC94792j/068b9c9a_stream2" \
       -c:v libx264 -preset veryfast -b:v 1500k \
       -c:a aac -b:a 128k \
       -f flv "rtmp://localhost:1935/live/test"
```

### 從本地視頻檔案串流
```bash
ffmpeg -re -i "本地視頻檔案.mp4" \
       -c:v libx264 -preset veryfast -b:v 1500k \
       -c:a aac -b:a 128k \
       -f flv "rtmp://localhost:1935/live/test"
```

### 從攝像頭串流
```bash
ffmpeg -f avfoundation -framerate 30 -video_size 1280x720 -i "0:0" \
       -c:v libx264 -preset veryfast -tune zerolatency -b:v 1500k \
       -c:a aac -b:a 128k \
       -f flv "rtmp://localhost:1935/live/test"
``` 