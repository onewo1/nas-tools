
FROM alpine:3.17 AS Builder

# 安装构建依赖并下载所需包
RUN apk add --no-cache --virtual .build-deps \
    libffi-dev \
    gcc \
    musl-dev \
    libxml2-dev \
    libxslt-dev \
    && apk add --no-cache $(echo $(wget --no-check-certificate -qO- https://raw.githubusercontent.com/onewo1/nas-tools50/master/package_list.txt)) \
    && ln -sf /usr/bin/python3 /usr/bin/python \
    && curl https://rclone.org/install.sh | bash \
    && if [ "$(uname -m)" = "x86_64" ]; then ARCH=amd64; elif [ "$(uname -m)" = "aarch64" ]; then ARCH=arm64; fi \
    && curl https://dl.min.io/client/mc/release/linux-${ARCH}/mc --create-dirs -o /usr/bin/mc \
    && chmod +x /usr/bin/mc \
    && pip install --upgrade pip setuptools wheel -i https://pypi.tuna.tsinghua.edu.cn/simple/ \
    && pip install cython -i https://pypi.tuna.tsinghua.edu.cn/simple/ \
    && pip install --ignore-installed packaging \
    && pip install -r https://raw.githubusercontent.com/onewo1/nas-tools50/master/requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple/ \
    && apk del --purge .build-deps \
    && rm -rf /tmp/* /root/.cache /var/cache/apk/*

# 复制根文件系统
COPY --chmod=755 ./docker/rootfs /

# 第二阶段：创建最终镜像
FROM scratch AS APP
COPY --from=Builder / /

# 设置环境变量
ENV S6_SERVICES_GRACETIME=30000 \
    S6_KILL_GRACETIME=60000 \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0 \
    S6_SYNC_DISKS=1 \
    HOME="/nt" \
    TERM="xterm" \
    PATH=${PATH}:/usr/lib/chromium \
    LANG="C.UTF-8" \
    TZ="Aia/Shanghai" \
    NASTOOL_CONFIG="/config/config.yaml" \
    NASTOOL_AUTO_UPDATE=false \
    NASTOOL_CN_UPDATE=true \
    NASTOOL_VERSION=master \
    PS1="\u@\h:\w \$ " \
    REPO_URL="http://github.com/onewo1/na-tool.git" \
    PYPI_MIRROR="http://pypi.tuna.tinghua.edu.cn/imple" \
    ALPINE_MIRROR="mirror.utc.edu.cn" \
    PUID=0 \
    PGID=0 \
    UMASK=000 \
    WORKDIR="/nas-tools"

# 复制项目文件并设置工作目录
COPY . /${WORKDIR}
WORKDIR ${WORKDIR}

# 初始化系统配置和用户权限
RUN addgroup -S nt -g 911 \
    && adduser -S nt -G nt -h ${HOME} -s /bin/bash -u 911 \
    && python_ver=$(python3 -V | awk '{print $2}') \
    && echo "${WORKDIR}/" > /usr/lib/python${python_ver%.*}/site-packages/nas-tools.pth \
    && echo 'f.inotify.max_uer_watche=5242880' >> /etc/sysctl.conf \
    && echo 'f.inotify.max_uer_intance=5242880' >> /etc/sysctl.conf \
    && echo "nt ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers \
    && cd third_party  \
    && git submodule init  \
    && git submodule update

# 暴露端口和挂载点
EXPOSE 3000
VOLUME ["/config"]

# 设置入口点
ENTRYPOINT [ "/init" ]

