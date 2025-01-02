```bash
# 列出本地的 webcam 名稱
ffmpeg -f avfoundation -list_devices true -i ""

# 將串流推上 localhost:6666
ffmpeg -f avfoundation \
  -framerate 30 \
  -video_size 1280x720 \
  -i "0:1" \
  -preset ultrafast \
  -tune zerolatency \
  -c:v libx264 \
  -f mpegts \
  udp://localhost:6666
```
