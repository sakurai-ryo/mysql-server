# `mysql:8.4`イメージと合わせたいため`oraclelinux:9`をベースに利用
# https://hub.docker.com/layers/library/mysql/8.4/images/sha256-a235c5bbb7235534ceca86d0dff98977aa854dd5b2ee507745458343d80454e4
FROM oraclelinux:9 AS builder

ENV MYSQL_VERSION=8.4.3

WORKDIR /usr/src

RUN dnf update -y

# MySQLのソースをCloneする（ヘッダーファイルなどが必要なため）
# 後のステップが失敗しやすいのでレイヤーキャッシュの関係で先にやっちゃう
RUN dnf install -y git
RUN git clone --depth 1 --branch 8.4 \
    https://github.com/mysql/mysql-server.git mysql-src

# ビルドに必要なパッケージをインストール
# !WARN: エラーメッセージに従って色々足したので細かい部分はあんま分かってない
RUN dnf install -y oracle-epel-release-el9 && \
    dnf config-manager --set-enabled ol9_codeready_builder
RUN dnf install -y \
    gcc-toolset-12-gcc \
    gcc-toolset-12-gcc-c++ \
    gcc-toolset-12-binutils \
    gcc-toolset-12-annobin-annocheck \
    gcc-toolset-12-annobin-plugin-gcc \
    cmake \
    openssl-devel \
    zlib-devel \
    ncurses-devel \
    libtirpc-devel \
    rpcgen \
    elfutils \
    bison


COPY CMakeLists.txt ha_toydb.cc ha_toydb.h /usr/src/mysql-src/storage/toydb/

WORKDIR /usr/src/build
RUN cmake /usr/src/mysql-src \
    -DDOWNLOAD_BOOST=1 \
    -DWITH_BOOST=/usr/src/boost \
    -DWITH_UNIT_TESTS=OFF

# Storage Engineのビルド
RUN cmake --build . --target toydb -- -j$(nproc)

# ==========================================================
# Stage 2: MySQL 8.4 + toydb plugin
# ==========================================================
FROM mysql:8.4

COPY --from=builder /usr/src/build/plugin_output_directory/ha_toydb.so \
    /usr/lib64/mysql/plugin/

# 初回起動時に自動でプラグインをインストール
RUN echo 'INSTALL PLUGIN toydb SONAME "ha_toydb.so";' \
    > /docker-entrypoint-initdb.d/install_toydb.sql

CMD ["mysqld", "--skip-log-bin"]
