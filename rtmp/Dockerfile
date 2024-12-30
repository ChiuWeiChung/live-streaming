# 使用 tiangolo/nginx-rtmp 作為基礎映像
FROM tiangolo/nginx-rtmp

# 更新包管理器並安裝 FFmpeg
RUN apt-get update && \
    apt-get install -y ffmpeg && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 複製 nginx 配置文件
COPY nginx.conf /etc/nginx/nginx.conf

# 複製 index.html 到 WWW 目錄
COPY index.html /www/

# 設置工作目錄
WORKDIR /www