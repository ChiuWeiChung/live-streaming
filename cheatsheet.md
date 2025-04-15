```bash
ffmpeg -rtsp_transport tcp  -i "rtsp://807e9439d5ca.entrypoint.cloud.wowza.com:1935/app-rC94792j/068b9c9a_stream2" \
       -c:v libx264 -preset veryfast -b:v 1500k \
       -c:a aac -b:a 128k \
       -f flv "rtmp://localhost:1935/live/test"
```       

```bash
ffmpeg -re -i https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8 \
       -c:v libx264 -preset veryfast -b:v 1500k \
       -c:a aac -b:a 128k \
       -f flv "rtmp://localhost:1935/live/test"
```