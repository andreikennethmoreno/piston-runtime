# STAGE 1: Build Isolate Sandbox
FROM buildpack-deps:buster AS isolate

# FIX: REWRITE SOURCES FOR EOL BUSTER
RUN sed -i 's/deb.debian.org/archive.debian.org/g' /etc/apt/sources.list && \
    sed -i 's|security.debian.org/debian-security|archive.debian.org/debian-security|g' /etc/apt/sources.list && \
    sed -i '/buster-updates/d' /etc/apt/sources.list

RUN apt-get update && \
    apt-get install -y --no-install-recommends git libcap-dev && \
    rm -rf /var/lib/apt/lists/* && \
    git clone https://github.com/envicutor/isolate.git /tmp/isolate/ && \
    cd /tmp/isolate && \
    git checkout af6db68042c3aa0ded80787fbb78bc0846ea2114 && \
    make -j$(nproc) install && \
    rm -rf /tmp/*

# STAGE 2: Main Piston API
FROM node:15.10.0-buster-slim

ENV DEBIAN_FRONTEND=noninteractive
RUN dpkg-reconfigure -p critical dash

# FIX: REWRITE SOURCES FOR EOL BUSTER
RUN sed -i 's/deb.debian.org/archive.debian.org/g' /etc/apt/sources.list && \
    sed -i 's|security.debian.org/debian-security|archive.debian.org/debian-security|g' /etc/apt/sources.list && \
    sed -i '/buster-updates/d' /etc/apt/sources.list

RUN apt-get update && \
    apt-get install -y libxml2 gnupg tar coreutils util-linux libc6-dev \
    binutils build-essential locales libpcre3-dev libevent-dev libgmp3-dev \
    libncurses6 libncurses5 libedit-dev libseccomp-dev rename procps python3 \
    libreadline-dev libblas-dev liblapack-dev libpcre3-dev libarpack2-dev \
    libfftw3-dev libglpk-dev libqhull-dev libqrupdate-dev libsuitesparse-dev \
    libsundials-dev libpcre2-dev libcap-dev && \
    rm -rf /var/lib/apt/lists/*

RUN useradd -M piston
COPY --from=isolate /usr/local/bin/isolate /usr/local/bin
COPY --from=isolate /usr/local/etc/isolate /usr/local/etc/isolate

RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && locale-gen

# Set main API directory
WORKDIR /piston_api

# 1. Install API dependencies
COPY ./api/package.json ./api/package-lock.json ./
RUN npm install
COPY ./api/src ./src

RUN sed -i 's/\r$//' ./src/docker-entrypoint.sh

# 2. Setup CLI correctly
WORKDIR /piston_api/cli
COPY ./cli/package.json ./cli/package-lock.json ./
RUN npm install
COPY ./cli ./

# 3. Create an Auto-Installer Runtime Script
# This starts the API in the background, waits for it to boot, installs languages, and holds the process open.
RUN echo '#!/bin/bash\n\
echo "--> Starting Piston API..."\n\
/piston_api/src/docker-entrypoint.sh &\n\
API_PID=$!\n\
echo "--> Waiting 10 seconds for API to initialize..."\n\
sleep 10\n\
echo "--> Installing Languages..."\n\
cd /piston_api/cli\n\
node index.js ppman install node=18.15.0 typescript=5.0.3 python=3.10.0 php=8.2.3 ruby=3.0.1 gcc=10.2.0 go=1.16.2 rust=1.68.2 mono=6.12.0 swift=5.3.3 java=15.0.2 kotlin=1.8.20\n\
echo "--> All languages installed! System ready."\n\
wait $API_PID\n\
' > /start.sh && chmod +x /start.sh

# Switch back to root to execute the script
WORKDIR /piston_api
CMD ["/start.sh"]
EXPOSE 2000/tcp