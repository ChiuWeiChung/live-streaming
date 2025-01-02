```bash
# In Local Machine
mpv \
  rtmp://localhost/live/bbb \
  --no-cache \
  --untimed \
  --no-demuxer-thread \
  --video-sync=audio \
  --vd-lavc-threads=1

# In Docker Container
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

## ffmpeg 指令解釋
* -re: 以實時速度讀取輸入文件，適用於推流場景。
* -i 'bbb_sunflower_1080p_30fps_normal.mp4': 指定輸入文件。
* -b:v 2500k: 設置影片 bit rate 為 2500 kbps，控制影片品質和大小。
* -c:v libx264: 使用 H.264 編碼器進行影片壓縮。
* -preset ultrafast: 設置編碼速度為最快，減少延遲（但可能降低壓縮效率）。
* -tune zerolatency: 為低延遲直播進行優化。
* -c:a aac: 使用 AAC 編碼器進行音頻壓縮。
* -f flv: 指定輸出格式為 FLV（適用於 RTMP 推流）。
* rtmp://host.docker.internal/live/bbb: 目標地址，將 stream 發送到 Docker 容器中的 RTMP 伺服器。

## mpv 指令解釋
* rtmp://localhost/live/bbb: 指定 RTMP stream 的播放地址。
* --no-cache: 禁用緩衝區，以減少播放延遲。
* --untimed: 忽略時間同步，用於直播流播放。
* --no-demuxer-thread: 禁用解復用的多線程處理，適合低延遲播放。
* --video-sync=audio: 將影片同步到音頻，減少音訊跟影片不同步的問題。
* --vd-lavc-threads=1: 限制影片解碼使用單線程，適合降低延遲的場景。