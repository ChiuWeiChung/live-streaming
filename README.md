```bash
# 列出本地的 webcam 名稱
ffmpeg -f avfoundation -list_devices true -i ""

# 監聽串流 (localhost:6666)
mpv --no-cache --profile=low-latency --video-sync=audio \
    --demuxer-max-bytes=256k \
    udp://localhost:6666

# 將串流推上 localhost:6666
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


# 監聽串流的參數
* udp://localhost:6666:
	* 指定要監聽的 UDP 流地址（來自 localhost 的 6666 端口）。
* --no-cache:
	* 禁用緩衝區，直接播放數據，減少延遲。
* --profile=low-latency:
	* 使用低延遲配置，以提高實時播放性能。
* --video-sync=audio:
	* 將視頻同步到音頻，減少音視頻不同步的問題。
* --demuxer-max-bytes=256k:
	* 設置最大緩衝大小為 256KB，適合低延遲播放。

# 輸出串流的參數
* 輸入相關參數
	* -f avfoundation:
		* 使用 macOS 的多媒體框架捕捉攝像頭和麥克風。
	* -framerate 30:
		* 設置捕捉幀率為每秒 30 幀。
	* -video_size 1280x720:
		* 設置捕捉視頻的分辨率為 1280x720（HD）。
	* -i "0:0":
		* 指定輸入設備：
		* 第一個視頻設備（如默認攝像頭）。
* 視頻編碼相關參數
	* -b:v 1000k:
		* 設置視頻比特率為 1000 kbps，適合低延遲流。
	* -c:v libx264:
		* 指定視頻編碼器為 H.264。
	* -preset ultrafast:
		* 使用 H.264 編碼器的最快預設，減少編碼延遲。
	* -tune zerolatency:
		* 調整編碼器以優化低延遲直播。
	* -g 30:
		* 設置 GOP（幀組）的大小為 30，意味著每 30 幀插入一個關鍵幀。
* 延遲優化參數
	* -fflags nobuffer:
		* 禁用內部緩衝區，減少延遲。
	* -flags low_delay:
		* 啟用低延遲編碼模式。
	* -flush_packets 1:
		* 強制在每個數據包後刷新輸出緩衝。
	* -max_delay 0:
		* 將最大延遲設置為零，確保最小延遲。
* 編碼器和輸出格式
	* -f mpegts:
		* 指定輸出格式為 MPEG-TS（傳輸流格式）。
	* udp://localhost:6666:
		* 將流推送到本地的 UDP 端口 6666。


## 會延遲的設定如下
```bash
# 監聽串流 (localhost:6666) 
mpv --no-cache  --video-sync=audio \
    udp://localhost:6666

# 將串流推上 localhost:6666
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