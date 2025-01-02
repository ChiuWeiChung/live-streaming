```bash
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

# 格式化說明
* 輸入文件：-i bbb_sunflower_1080p_30fps_normal.mp4
* 
# 影片編碼設置：
* -b:v 2500k：設置比特率為 2500kbps。
* -c:v libx264：使用 H.264 編碼器。
* -preset ultrafast：使用最快的編碼模式。
* -tune zerolatency：優化低延遲。

# 輸出格式：
* -f mpegts：使用 MPEG-TS 格式。
* -fifo_size 1000000：設置 FIFO 緩衝大小。
* 輸出地址：udp://host.docker.internal:6666?overrun_nonfatal=1。