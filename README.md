# 直播串流系統 (Live Streaming System)

這個專案是一個簡單的直播串流系統，主要功能有兩個部分：

1. **RTMP 串流伺服器**：使用 Docker 容器部署了一個基於 Nginx 的 RTMP 伺服器，可以接收和分發直播串流。

2. **串流工具**：提供了兩個主要命令：
   - 使用 ffmpeg 從您的攝像頭和麥克風捕獲視頻和音頻，並將其推送到 RTMP 伺服器
   - 使用 mpv 播放器從 RTMP 伺服器接收和播放串流

整個設置允許您在本地主機上進行直播串流測試或開發。Docker 容器暴露了多個端口（1935 用於 RTMP，8080 用於 HTTP）。

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

### 播放 RTMP 串流 (Play RTMP Stream)
```bash
mpv \
  rtmp://localhost/live/stream \
  --no-cache \
  --untimed \
  --vd-lavc-threads=2 \
  --hwdec=auto-safe \
  --video-sync=display-resample \
  --audio-buffer=0.5
```  

### 推送攝像頭和麥克風串流到 RTMP 伺服器 (Push Camera/Mic to RTMP Server)
```
ffmpeg \
  -f avfoundation \
  -framerate 30 \
  -video_size 1280x720 \
  -i "0:0" \
  -b:v 1500k \
  -preset superfast \
  -tune zerolatency \
  -c:v libx264 \
  -c:a aac \
  -b:a 128k \
  -f flv \
  rtmp://localhost/live/stream
```

## 指令解釋

### mpv 指令解釋
* rtmp://localhost/live/stream
	* 指定播放的 RTMP 流地址：
	* localhost 表示本地 RTMP 伺服器。
	* /live/stream 是應用名稱和流名稱。
* --no-cache
	* 禁用播放器的緩衝機制，直接播放接收到的數據，減少延遲。
	* 適合用於直播或低延遲場景。
* --untimed
	* 忽略時間戳同步：
	* 將音視頻流無需嚴格按照時間戳播放，適合處理非同步的實時流。
* --vd-lavc-threads=2
	* 設置視頻解碼使用的線程數為 2：
	* 多線程解碼可以提高播放性能，但線程數過高可能導致同步問題。
	* 默認設置為 1，適合低延遲流。
* --hwdec=auto-safe
	* 啟用硬件解碼：
	* 自動選擇安全的硬件解碼方式（如果硬件支持）。
	* 減少 CPU 負載，提高解碼效率。
* --video-sync=display-resample
	* 將視頻同步到顯示器的刷新率，並對音頻進行重採樣：
	* 減少視頻卡頓和音視頻不同步的問題。
* --audio-buffer=0.5
	* 設置音頻緩衝區大小為 0.5 秒：
	* 增加音頻緩衝區有助於減少音頻設備的緩衝不足（Audio device underrun detected）問題。

### ffmpeg 指令解釋
* -f avfoundation
	* 指定輸入格式為 avfoundation（適用於 macOS 的多媒體框架，用於捕獲攝像頭和麥克風）。
* -framerate 30
	* 設置捕獲幀率為 30 幀每秒。
* -video_size 1280x720
	* 設置視頻分辨率為 1280x720（HD 分辨率）。
* -i "0:0"
	* 指定輸入設備：
	* 0 表示第一個視頻輸入設備（通常是默認攝像頭）。
	* 0 表示第一個音頻輸入設備（通常是默認麥克風）。
* -b:v 1500k
	* 設置視頻比特率為 1500 kbps，用於平衡視頻質量和帶寬。
* -preset superfast
	* 使用 libx264 編碼器的快速預設：
	* ultrafast 更快，但壓縮效率低。
	* superfast 是一種折中方案，速度快且具有更好的壓縮效率。
* -tune zerolatency
	* 用於低延遲場景的編碼優化（適合直播）。
* -c:v libx264
	* 指定視頻編碼器為 libx264（H.264 編碼）。
* -c:a aac
	* 指定音頻編碼器為 aac（先進音頻編碼）。
* -b:a 128k
	* 設置音頻比特率為 128 kbps，平衡音頻質量和大小。
* -f flv
	* 指定輸出格式為 FLV（Flash Video Format，適用於 RTMP 推流）。
* rtmp://localhost/live/stream
	* 指定 RTMP 推流目標地址：
	* localhost 表示本地 RTMP 伺服器。
	* /live/stream 是推流的應用名稱和流名稱。