# RTMP 串流影片測試系統

這個專案是一個 RTMP 串流影片測試系統，主要提供以下功能：

1. **RTMP 串流伺服器**：使用 Docker 容器部署了一個基於 Nginx 的 RTMP 伺服器，可以接收和分發直播串流。

2. **影片串流測試工具**：
   - 在 Docker 容器內，用 ffmpeg 將預先存在的 Big Buck Bunny 影片檔案 (bbb_sunflower_1080p_30fps_normal.mp4) 推送到 RTMP 伺服器
   - 在本地機器上，使用 mpv 播放器接收和播放這個串流

整個設置非常適合測試和開發 RTMP 串流服務，尤其是在不需要實時攝像頭影像的情況下。

## 設置說明

### 前置需求
- Docker 和 Docker Compose
- ffmpeg (用於推送串流)
- mpv 播放器 (用於播放串流)

### 啟動 RTMP 伺服器
```bash
docker-compose up -d
```

這將啟動一個 RTMP 伺服器，監聽以下端口：
- 1935: RTMP 協議
- 8080: HTTP 服務 (可用於網頁播放器)

## 使用說明

### 播放 RTMP 串流 (本地機器)
```bash
mpv \
  rtmp://localhost/live/bbb \
  --no-cache \
  --untimed \
  --no-demuxer-thread \
  --video-sync=audio \
  --vd-lavc-threads=1
```

### 推送影片串流到 RTMP 伺服器 (Docker 容器內)
```bash
ffmpeg \
  -re \
  -i 'bbb_sunflower_1080p_30fps_normal.mp4' \
  -b:v 2500k \
  -c:v libx264 \
  -preset ultrafast \
  -tune zerolatency \
  -c:a aac \
  -f flv \
  rtmp://host.docker.internal/live/bbb
```

## 指令解釋

### ffmpeg 指令解釋
* -re: 以實時速度讀取輸入文件，適用於推流場景。
* -i 'bbb_sunflower_1080p_30fps_normal.mp4': 指定輸入文件。
* -b:v 2500k: 設置影片 bit rate 為 2500 kbps，控制影片品質和大小。
* -c:v libx264: 使用 H.264 編碼器進行影片壓縮。
* -preset ultrafast: 設置編碼速度為最快，減少延遲（但可能降低壓縮效率）。
* -tune zerolatency: 為低延遲直播進行優化。
* -c:a aac: 使用 AAC 編碼器進行音頻壓縮。
* -f flv: 指定輸出格式為 FLV（適用於 RTMP 推流）。
* rtmp://host.docker.internal/live/bbb: 目標地址，將 stream 發送到 Docker 容器中的 RTMP 伺服器。

### mpv 指令解釋
* rtmp://localhost/live/bbb: 指定 RTMP stream 的播放地址。
* --no-cache: 禁用緩衝區，以減少播放延遲。
* --untimed: 忽略時間同步，用於直播流播放。
* --no-demuxer-thread: 禁用解復用的多線程處理，適合低延遲播放。
* --video-sync=audio: 將影片同步到音頻，減少音訊跟影片不同步的問題。
* --vd-lavc-threads=1: 限制影片解碼使用單線程，適合降低延遲的場景。