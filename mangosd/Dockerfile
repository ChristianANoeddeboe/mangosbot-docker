# Download base image ubuntu
FROM ubuntu:22.04 AS base

ENV CMAKE_VERSION=3.24.2

RUN echo "ENV_CMAKE_VERSION=$CMAKE_VERSION"

# Update Ubuntu Software repository
RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -q -y \
    # Essential for building with cmake
    build-essential \
    gcc \
    g++ \
    automake \
    git-core \
    autoconf \
    make \
    patch \
    libmysql++-dev \
    mysql-server \
    libtool \
    libssl-dev \
    libssl3 \
    grep \
    binutils \
    zlib1g-dev \
    libbz2-dev \
    cmake \
    libboost-dev \
    libboost-filesystem-dev \ 
    libboost-program-options-dev \
    libboost-regex-dev \
    libboost-system-dev \
    libboost-thread-dev \
    libboost-all-dev \
    # Essential for working with files
    wget \
    unzip \
    ca-certificates \
    # Essential for running the server
    libmysqlclient21 \
    libreadline8 \
    libncurses6 \ 
    libc6 \
    libcurl4-openssl-dev \
    libpthread-stubs0-dev \
    openssl \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Temporary fix for broken GCC
RUN apt-get update && apt-get -y install g++-12 && \
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 12 --slave /usr/bin/g++ g++ /usr/bin/g++-12

# Prepare new cmake
RUN cd /tmp && wget --no-check-certificate https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz && \
    tar -xvzf cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz && \
    mv /tmp/cmake-${CMAKE_VERSION}-linux-x86_64 /opt/cmake && \
    ln -s /opt/cmake/bin/* /usr/local/bin/ && \
    rm -f /tmp/cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz

FROM base AS builder

ARG CORE_COMMIT_HASH
ARG DB_COMMIT_HASH
ARG BUILD_AHBOT=OFF
ARG BUILD_PLAYERBOT=OFF
ARG BUILD_EXTRACTORS=OFF
ARG CMAKE_FLAGS=""
# Write out build arguments
RUN echo "CORE_COMMIT_HASH=$CORE_COMMIT_HASH" && \
    echo "DB_COMMIT_HASH=$DB_COMMIT_HASH" && \
    echo "BUILD_AHBOT=$BUILD_AHBOT" && \
    echo "BUILD_PLAYERBOT=$BUILD_PLAYERBOT" && \
    echo "BUILD_EXTRACTORS=$BUILD_EXTRACTORS" && \
    echo "CMAKE_FLAGS=$CMAKE_FLAGS"

# Set Timezone
ENV TZ=Europe/Amsterdam
ENV ENV_CORE_COMMIT_HASH=$CORE_COMMIT_HASH
ENV ENV_DB_COMMIT_HASH=$DB_COMMIT_HASH
ENV ENV_CMAKE_FLAGS=$CMAKE_FLAGS

# Write out environment variables
RUN echo "ENV_CORE_COMMIT_HASH=$ENV_CORE_COMMIT_HASH" && \
    echo "ENV_DB_COMMIT_HASH=$ENV_DB_COMMIT_HASH" && \
    echo "ENV_BUILD_AHBOT=$BUILD_AHBOT" && \
    echo "ENV_BUILD_PLAYERBOT=$BUILD_PLAYERBOT" && \
    echo "ENV_CMAKE_FLAGS=$ENV_CMAKE_FLAGS"

# Prepare Sources
RUN mkdir -p /opt/cmangos && \
    git clone https://github.com/ChristianANoeddeboe/mangos-classic.git /opt/cmangos && \
    git clone https://github.com/ChristianANoeddeboe/playerbots.git /opt/cmangos/src/modules/PlayerBots && \
    echo "Core Version: ${ENV_CORE_COMMIT_HASH}" && \
    if [ "$ENV_CORE_COMMIT_HASH" != "HEAD" ]; then echo "Switching to Commit: ${ENV_CORE_COMMIT_HASH}\n"; cd /opt/cmangos; git checkout ${ENV_CORE_COMMIT_HASH}; fi

# Print out CMAKE flags
RUN echo "Final CMAKE_FLAGS before build: $CMAKE_FLAGS"

# Build Software
RUN mkdir -p /opt/cmangos/logs /opt/cmangos/build

RUN cd /opt/cmangos/build && \
    cmake /opt/cmangos \
    -DCMAKE_EXPORT_COMPILE_COMMANDS=ON \
    -DCMAKE_VERBOSE_MAKEFILE=ON \
    # -DCMAKE_BUILD_TYPE=RelWithDebInfo \
    # -DCMAKE_C_FLAGS="-Wno-switch -Wno-format -Wno-format-security -Wno-format-extra-args -Wno-unused-value -O0 -g" \
    # -DCMAKE_CXX_FLAGS="-Wno-switch -Wno-format -Wno-format-security -Wno-format-extra-args -Wno-unused-value -O0 -g" \
    -DCMAKE_BUILD_TYPE=Release \
    # -DPCH=1 \
    -DDEBUG=0 \
    -DCMAKE_INSTALL_PREFIX=/opt/cmangos \
    -DBUILD_EXTRACTORS=${BUILD_EXTRACTORS} \
    -DBUILD_AHBOT=${BUILD_AHBOT} \
    -DBUILD_PLAYERBOTS=${BUILD_PLAYERBOT} \
    -DBUILD_LOGIN_SERVER=OFF \
    -DBUILD_GAME_SERVER=ON \
    -DBUILD_ACHIEVEMENTS=OFF \
    -DBUILD_GIT_ID=OFF \
    -DBUILD_SCRIPTDEV=ON \
    -DPCH=ON \
    $ENV_CMAKE_FLAGS
# Release with Debug info build
# Memory leak debugger build
#cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_C_FLAGS="-fsanitize=address,leak -fsanitize-recover=address -O0 -g -Wno-switch -Wno-format -Wno-format-security -Wno-format-extra-args -Wno-unused-value" -DCMAKE_CXX_FLAGS="-fsanitize=address,leak -fsanitize-recover=address -O0 -g -Wno-switch -Wno-format -Wno-format-security -Wno-format-extra-args -Wno-unused-value" -DCMAKE_MODULE_LINKER_FLAGS="-fsanitize=address,leak" -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_INSTALL_PREFIX=/opt/cmangos -DBUILD_LOGIN_SERVER=OFF -DBUILD_GAME_SERVER=ON -DBUILD_ACHIEVEMENTS=ON -DBUILD_AHBOT=ON -DBUILD_GIT_ID=OFF -DBUILD_IKE3_BOTS=ON -DBUILD_PLAYERBOT=ON -DBUILD_SCRIPTDEV=ON -DPCH=ON && \
RUN cd /opt/cmangos/build && make VERBOSE=1 -j$(nproc) && make install

# Save current repository version used into a file
RUN cd /opt/cmangos && \
    git rev-parse HEAD > /opt/cmangos/bin/core_version.txt && \
    cd /opt/cmangos/src/modules/PlayerBots && \
    git rev-parse HEAD > /opt/cmangos/bin/mangosbot_version.txt

# Download base image ubuntu
FROM ubuntu:22.04 AS runner

# Install dependencies for running CMaNGOSd.
RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -q -y \
    binutils \
    libboost-all-dev \
    libc6 \
    libcurl4 \
    libmysql++3v5 \
    libpthread-stubs0-dev \
    libssl3 \
    libtool \
    openssl \
    pkg-config \
    tzdata \
    unrar-free \
    unzip \
    zlib1g-dev \
    openssh-server \
    curl \
    git-core \
    ca-certificates \
    mysql-server \
    gdb && \
    rm -rf /var/lib/apt/lists/*

# Copy over data
COPY --from=builder /opt/cmangos /opt/cmangos
RUN mv /opt/cmangos/bin /opt/cmangos/bin2

# Volume configuration
VOLUME ["/opt/cmangos/etc"]

# Fetch build arguments
ARG CORE_COMMIT_HASH=HEAD \
    DB_COMMIT_HASH=HEAD \
    BUILD_AHBOT=ON \
    BUILD_PLAYERBOT=OFF

# Env Variables
ENV TZ=Europe/Amsterdam \
    ENV_CORE_COMMIT_HASH=$CORE_COMMIT_HASH \
    ENV_DB_COMMIT_HASH=$DB_COMMIT_HASH \
    ENV_BUILD_AHBOT=$BUILD_AHBOT \
    ENV_BUILD_PLAYERBOT=$BUILD_PLAYERBOT \
    CHARACTERS_DB=classiccharacters \
    LOGS_DB=classiclogs \
    MANGOSD_DB=classicmangos \
    REALMD_DB=classicrealmd \
    PLAYERBOTS_DB=classicplayerbots \
    DB_USER=mangos \
    DB_PASS=mangos \
    DB_PORT=3306 \
    DB_SERVER=database \
    DB_ROOT_USER=root \
    DB_ROOT_PASS=mangos

# Copy over init scripts and make them executable
COPY scripts/mangosd.sh /opt/cmangos/scripts/mangosd.sh
COPY scripts/wait-for-it.sh /opt/cmangos/scripts/wait-for-it.sh
RUN chmod +x /opt/cmangos/scripts/mangosd.sh && chmod +x /opt/cmangos/scripts/wait-for-it.sh

# Prepare tini init
ENV TINI_VERSION=v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]
CMD ["/opt/cmangos/scripts/mangosd.sh", "start"]

# Expose ports
EXPOSE 8085
