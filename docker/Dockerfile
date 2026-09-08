# 固定 alpine 3.17：Python 3.10 与 requirements.txt 的 2022 年锁定版本兼容
FROM alpine:3.17
# 时区等环境变量提前声明，使构建阶段生效
ENV LANG="C.UTF-8" \
    TZ="Asia/Shanghai" \
    NASTOOL_CONFIG="/config/config.yaml" \
    NASTOOL_AUTO_UPDATE=false \
    NASTOOL_CN_UPDATE=true \
    NASTOOL_VERSION=master \
    PS1="\u@\h:\w \$ " \
    REPO_URL="https://github.com/NAStool/nas-tools.git" \
    PYPI_MIRROR="https://pypi.tuna.tsinghua.edu.cn/simple" \
    ALPINE_MIRROR="mirrors.ustc.edu.cn" \
    PUID=0 \
    PGID=0 \
    UMASK=000 \
    WORKDIR="/nas-tools"
# 从本地拷贝依赖清单（原 jxxghp/nas-tools 仓库已不可在线获取）
COPY package_list.txt requirements.txt /tmp/
RUN apk add --no-cache libffi-dev \
    && apk add --no-cache $(cat /tmp/package_list.txt) \
    && ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo "${TZ}" > /etc/timezone \
    && ln -sf /usr/bin/python3 /usr/bin/python \
    && curl https://rclone.org/install.sh | bash \
    && if [ "$(uname -m)" = "x86_64" ]; then ARCH=amd64; elif [ "$(uname -m)" = "aarch64" ]; then ARCH=arm64; fi \
    && curl https://dl.min.io/client/mc/release/linux-${ARCH}/mc --create-dirs -o /usr/bin/mc \
    && chmod +x /usr/bin/mc \
    && pip install --upgrade pip setuptools wheel -i ${PYPI_MIRROR} \
    && pip install cython -i ${PYPI_MIRROR} \
    && pip install -r /tmp/requirements.txt -i ${PYPI_MIRROR} \
    && apk del libffi-dev \
    && npm install pm2 -g --registry=https://registry.npmmirror.com \
    && rm -rf /tmp/* /root/.cache /var/cache/apk/*
WORKDIR ${WORKDIR}
RUN python_ver=$(python3 -V | awk '{print $2}') \
    && echo "${WORKDIR}/" > /usr/lib/python${python_ver%.*}/site-packages/nas-tools.pth \
    && echo 'fs.inotify.max_user_watches=524288' >> /etc/sysctl.conf \
    && echo 'fs.inotify.max_user_instances=524288' >> /etc/sysctl.conf \
    && git config --global pull.ff only
# 拷贝本地源码替代 git clone（原仓库已不可在线获取）
COPY app ${WORKDIR}/app
COPY web ${WORKDIR}/web
COPY config ${WORKDIR}/config
COPY docker ${WORKDIR}/docker
COPY db_scripts ${WORKDIR}/db_scripts
COPY third_party ${WORKDIR}/third_party
COPY patch ${WORKDIR}/patch
COPY run.py config.py log.py version.py check_config.py build_sites.py dbscript_gen.py ${WORKDIR}/
COPY requirements.txt third_party.txt package_list.txt ${WORKDIR}/
# 豆瓣防盗链修复 modify by vv 20260908 starting
RUN git apply patch/douban-image-fix.patch
# 豆瓣防盗链修复 modify by vv 20260908 ending  
EXPOSE 3000
VOLUME ["/config"]
ENTRYPOINT ["/nas-tools/docker/entrypoint.sh"]