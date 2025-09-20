FROM alpine

# 设置时区环境变量
ENV TZ="Asia/Shanghai"

# 安装基础依赖
RUN apk add --no-cache \
    python3 \
    py3-pip \
    python3-dev \
    gcc \
    musl-dev \
    libffi-dev \
    wget \
    curl \
    npm \
    git \
    && ln -sf /usr/bin/python3 /usr/bin/python

# 下载并安装系统依赖包
RUN apk add --no-cache $(echo $(wget --no-check-certificate -qO- https://raw.githubusercontent.com/felix-ls/nas-tools/master/package_list.txt))

# 设置时区
RUN ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo "${TZ}" > /etc/timezone

# 安装rclone
RUN curl https://rclone.org/install.sh | bash

# 安装minio客户端
RUN if [ "$(uname -m)" = "x86_64" ]; then ARCH=amd64; elif [ "$(uname -m)" = "aarch64" ]; then ARCH=arm64; fi \
    && curl https://dl.min.io/client/mc/release/linux-${ARCH}/mc --create-dirs -o /usr/bin/mc \
    && chmod +x /usr/bin/mc

# 创建虚拟环境
RUN python -m venv /app/venv
ENV PATH="/app/venv/bin:$PATH"

# 升级pip并安装Python依赖
RUN pip install --upgrade pip setuptools wheel \
    && pip install cython \
    && pip install -r https://raw.githubusercontent.com/felix-ls/nas-tools/master/requirements.txt

# 清理
RUN apk del libffi-dev python3-dev gcc musl-dev \
    && npm install pm2 -g \
    && rm -rf /tmp/* /root/.cache /var/cache/apk/*

# 设置环境变量
ENV LANG="C.UTF-8" \
    NASTOOL_CONFIG="/config/config.yaml" \
    NASTOOL_AUTO_UPDATE=true \
    NASTOOL_CN_UPDATE=true \
    NASTOOL_VERSION=master \
    PS1="\u@\h:\w \$ " \
    REPO_URL="https://github.com/felix-ls/nas-tools.git" \
    PYPI_MIRROR="https://pypi.tuna.tsinghua.edu.cn/simple" \
    ALPINE_MIRROR="mirrors.ustc.edu.cn" \
    PUID=0 \
    PGID=0 \
    UMASK=000 \
    WORKDIR="/nas-tools"

WORKDIR ${WORKDIR}

RUN python_ver=$(python3 -V | awk '{print $2}') \
    && echo "${WORKDIR}/" > /usr/lib/python${python_ver%.*}/site-packages/nas-tools.pth \
    && echo 'fs.inotify.max_user_watches=524288' >> /etc/sysctl.conf \
    && echo 'fs.inotify.max_user_instances=524288' >> /etc/sysctl.conf \
    && git config --global pull.ff only \
    && git clone -b master ${REPO_URL} ${WORKDIR} --depth=1 --recurse-submodule \
    && git config --global --add safe.directory ${WORKDIR}

# 确保虚拟环境路径在环境变量中
ENV PATH="/app/venv/bin:$PATH"

EXPOSE 3000
VOLUME ["/config"]
ENTRYPOINT ["/nas-tools/docker/entrypoint.sh"]
