# UDP 串流影片測試系統

這個專案是一個 UDP 串流影片測試系統，主要提供以下功能：

1. **Docker 容器環境**：使用 Docker 容器作為影片串流的來源環境，在容器內將影片轉換為 UDP 串流。

2. **影片串流測試工具**：
   - 在 Docker 容器內，用 ffmpeg 將預先存在的 Big Buck Bunny 影片檔案 (bbb_sunflower_1080p_30fps_normal.mp4) 轉為 UDP 串流
   - 在本地機器上，使用 mpv 播放器接收和播放 UDP 串流

整個設置非常適合測試和開發低延遲 UDP 視頻串流服務，比 RTMP 提供更低的傳輸延遲。

## 設置說明

### 前置需求
- Docker 和 Docker Compose
- ffmpeg (用於推送串流)
- mpv 播放器 (用於播放串流)

### 啟動容器環境
```bash
docker-compose up -d
```

這將啟動一個 Docker 容器，開放以下端口：
- 1935: RTMP 協議 (此分支未使用)
- 8080: HTTP 服務 (此分支未使用)
- 6666: UDP 串流接收/發送

## 使用說明

### 在本地機器接收 UDP 串流
```bash
# 在本地機器執行
mpv udp://@:6666
```

### 從 Docker 容器發送 UDP 串流
```bash
# 在 Docker 容器內執行
ffmpeg \
  -i bbb_sunflower_1080p_30fps_normal.mp4 \
  -b:v 2500k \
  -c:v libx264 \
  -preset ultrafast \
  -tune zerolatency \
  -f mpegts \
  -fifo_size 1000000 \
  udp://host.docker.internal:6666?overrun_nonfatal=1
```

## 指令解釋

### ffmpeg UDP 串流指令解釋

#### 輸入設置
* -i bbb_sunflower_1080p_30fps_normal.mp4：指定輸入文件。

#### 影片編碼設置
* -b:v 2500k：設置比特率為 2500kbps，控制影片品質和大小。
* -c:v libx264：使用 H.264 編碼器進行影片壓縮。
* -preset ultrafast：使用最快的編碼模式，盡可能減少延遲。
* -tune zerolatency：為低延遲場景優化編碼。

#### 輸出格式
* -f mpegts：使用 MPEG-TS (Transport Stream) 格式，適合 UDP 傳輸。
* -fifo_size 1000000：設置 FIFO 緩衝大小，避免緩衝不足。
* udp://host.docker.internal:6666?overrun_nonfatal=1：
  * 目標 UDP 地址，指向宿主機的 6666 端口
  * overrun_nonfatal=1 參數允許在網絡擁塞時繼續傳輸，不會因緩衝區溢出而終止。

### mpv UDP 接收指令解釋
* udp://@:6666：指定 UDP 串流接收地址和端口。
  * @ 符號表示監聽所有網絡接口
  * 6666 是接收端口