# Download base image ubuntu
FROM ubuntu:22.04 AS builder

# Fetch build arguments
ARG CORE_COMMIT_HASH=HEAD \
    DB_COMMIT_HASH=HEAD \
    BUILD_AHBOT=OFF \
    BUILD_PLAYERBOT=OFF \
    BUILD_EXTRACTORS=OFF

# Write out build arguments
RUN echo "CORE_COMMIT_HASH=$CORE_COMMIT_HASH" && \
    echo "DB_COMMIT_HASH=$DB_COMMIT_HASH" && \
    echo "BUILD_AHBOT=$BUILD_AHBOT" && \
    echo "BUILD_PLAYERBOT=$BUILD_PLAYERBOT" && \
    echo "BUILD_EXTRACTORS=$BUILD_EXTRACTORS"

# Set Timezone
ENV TZ=Europe/Amsterdam \
    ENV_CORE_COMMIT_HASH=$CORE_COMMIT_HASH \
    ENV_DB_COMMIT_HASH=$DB_COMMIT_HASH \
    CMAKE_VERSION=3.24.2

# Write out environment variables
RUN echo "ENV_CORE_COMMIT_HASH=$ENV_CORE_COMMIT_HASH" && \
    echo "ENV_DB_COMMIT_HASH=$ENV_DB_COMMIT_HASH"

# Update Ubuntu Software repository
RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -q -y \
    # Essential for building with cmake
    build-essential gcc g++ automake git-core autoconf make patch libmysql++-dev mysql-server libtool libssl-dev grep binutils zlib1g-dev libbz2-dev cmake libboost-all-dev \
    # Essential for working with files
    wget unzip ca-certificates \
    # autoconf \
    # automake \
    # binutils \
    # build-essential \
    # ca-certificates \
    # cmake \
    # cpp \
    # g++ \
    # gcc \
    # #	  clang \
    # git-core \
    # libboost-dev \
    # libboost-filesystem-dev \ 
    # libboost-program-options-dev \
    # libboost-regex-dev \
    # libboost-system-dev \
    # libboost-thread-dev \
    # libc6 \
    # libcurl4-openssl-dev \
    # libmysql++-dev \
    # libpthread-stubs0-dev \
    # libssl-dev \
    # libtool \
    # make \
    # openssl \
    # patch \
    # pkg-config \
    # unrar-free \
    # unzip \
    # wget \
    # zlibc && \
    && rm -rf /var/lib/apt/lists/*

# Temporary fix for broken GCC
RUN apt-get update && apt-get -y install g++-12 && \
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 12 --slave /usr/bin/g++ g++ /usr/bin/g++-12

# Prepare new cmake
RUN cd /tmp && wget --no-check-certificate https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz && \
    tar -xvzf cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz && \
    mv /tmp/cmake-${CMAKE_VERSION}-linux-x86_64 /opt/cmake && \
    ln -s /opt/cmake/bin/* /usr/local/bin/ && \
    rm -f /tmp/cmake-${CMAKE_VERSION}-linux-x86_64.tar.gz

# Prepare Sources
RUN mkdir -p /opt/cmangos && \
    git clone https://github.com/cmangos/mangos-classic.git /opt/cmangos && \
    git clone https://github.com/cmangos/playerbots.git /opt/cmangos/src/modules/Bots && \
    echo "Core Version: ${ENV_CORE_COMMIT_HASH}" && \
    if [ "$ENV_CORE_COMMIT_HASH" != "HEAD" ]; then echo "Switching to Commit: ${ENV_CORE_COMMIT_HASH}\n"; cd /opt/cmangos; git checkout ${ENV_CORE_COMMIT_HASH}; fi

# Build Software
RUN mkdir -p /opt/cmangos/logs /opt/cmangos/build

RUN cd /opt/cmangos/build && \
    # Release build
    cmake /opt/cmangos -DCMAKE_BUILD_TYPE=Release \ 
    # -DCMAKE_C_FLAGS="-Wno-switch -Wno-format -Wno-format-security -Wno-format-extra-args -Wno-unused-value" \
    # -DCMAKE_CXX_FLAGS="-Wno-switch -Wno-format -Wno-format-security -Wno-format-extra-args -Wno-unused-value" \
    -DCMAKE_INSTALL_PREFIX=/opt/cmangos \
    -DBUILD_EXTRACTORS=${BUILD_EXTRACTORS} \
    -DBUILD_LOGIN_SERVER=OFF \
    -DBUILD_GAME_SERVER=ON \
    -DBUILD_ACHIEVEMENTS=OFF \
    -DBUILD_AHBOT=${BUILD_AHBOT} \
    -DBUILD_GIT_ID=OFF \
    -DBUILD_PLAYERBOT=${BUILD_PLAYERBOT} \
    -DBUILD_SCRIPTDEV=ON \
    -DPCH=ON
# Release with Debug info build
# cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_C_FLAGS="-Wno-switch -Wno-format -Wno-format-security -Wno-format-extra-args -Wno-unused-value -O0 -g" -DCMAKE_CXX_FLAGS="-Wno-switch -Wno-format -Wno-format-security -Wno-format-extra-args -Wno-unused-value -O0 -g" -DCMAKE_INSTALL_PREFIX=/opt/cmangos -DPCH=1 -DDEBUG=0 -DBUILD_PLAYERBOTS=ON -DBUILD_AHBOT=${ENV_BUILD_AHBOT} -DBUILD_GIT_ID=ON -DBUILD_MODULES=ON -DBUILD_MODULE_ACHIEVEMENTS=OFF -DBUILD_MODULE_DUALSPEC=ON -DBUILD_MODULE_HARDCORE=OFF -DBUILD_MODULE_IMMERSIVE=ON -DBUILD_MODULE_TRANSMOG=ON -DBUILD_MODULE_BARBER=OFF && \
# Memory leak debugger build
#cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_C_FLAGS="-fsanitize=address,leak -fsanitize-recover=address -O0 -g -Wno-switch -Wno-format -Wno-format-security -Wno-format-extra-args -Wno-unused-value" -DCMAKE_CXX_FLAGS="-fsanitize=address,leak -fsanitize-recover=address -O0 -g -Wno-switch -Wno-format -Wno-format-security -Wno-format-extra-args -Wno-unused-value" -DCMAKE_MODULE_LINKER_FLAGS="-fsanitize=address,leak" -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_INSTALL_PREFIX=/opt/cmangos -DBUILD_LOGIN_SERVER=OFF -DBUILD_GAME_SERVER=ON -DBUILD_ACHIEVEMENTS=ON -DBUILD_AHBOT=ON -DBUILD_GIT_ID=OFF -DBUILD_IKE3_BOTS=ON -DBUILD_PLAYERBOT=ON -DBUILD_SCRIPTDEV=ON -DPCH=ON && \
RUN cd /opt/cmangos/build && make -j$(nproc) && make install

# Save current repository version used into a file
RUN cd /opt/cmangos && \
    git rev-parse HEAD > /opt/cmangos/bin/core_version.txt && \
    cd /opt/cmangos/src/modules/Bots && \
    git rev-parse HEAD > /opt/cmangos/bin/mangosbot_version.txt

# Download base image ubuntu
FROM ubuntu:20.04

# Install dependencies for running CMaNGOSd.
RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -q -y \
    binutils \
    libboost-dev \
    libboost-filesystem1.71.0 \
    libboost-program-options1.71.0 \
    libboost-regex1.71.0 \
    libboost-system1.71.0 \
    libboost-thread1.71.0 \
    libc6 \
    libcurl4 \
    libmysql++3v5 \
    libpthread-stubs0-dev \
    libssl1.1 \
    libtool \
    openssl \
    pkg-config \
    tzdata \
    unrar-free \
    unzip \
    zlibc \
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
