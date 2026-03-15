# STAGE 1: Build Isolate Sandbox
FROM buildpack-deps:buster AS isolate

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

RUN sed -i 's/deb.debian.org/archive.debian.org/g' /etc/apt/sources.list && \
    sed -i 's|security.debian.org/debian-security|archive.debian.org/debian-security|g' /etc/apt/sources.list && \
    sed -i '/buster-updates/d' /etc/apt/sources.list

RUN apt-get update && \
    apt-get install -y libxml2 gnupg tar coreutils util-linux libc6-dev \
    binutils build-essential locales libpcre3-dev libevent-dev libgmp3-dev \
    libncurses6 libncurses5 libedit-dev libseccomp-dev rename procps python3 \
    libreadline-dev libblas-dev liblapack-dev libpcre3-dev libarpack2-dev \
    libfftw3-dev libglpk-dev libqhull-dev libqrupdate-dev libsuitesparse-dev \
    libsundials-dev libpcre2-dev libcap-dev curl && \
    rm -rf /var/lib/apt/lists/*

RUN useradd -M piston
COPY --from=isolate /usr/local/bin/isolate /usr/local/bin
COPY --from=isolate /usr/local/etc/isolate /usr/local/etc/isolate

RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && locale-gen

WORKDIR /piston_api

# Install API dependencies
COPY ./api/package.json ./api/package-lock.json ./
RUN npm install
COPY ./api/src ./src

# Setup CLI
WORKDIR /piston_api/cli
COPY ./cli/package.json ./cli/package-lock.json ./
RUN npm install
COPY ./cli ./

# CREATE BYPASS START SCRIPT
RUN echo '#!/bin/bash\n\
echo "--> Starting Piston API (Bypass Mode)..."\n\
cd /piston_api/src && node index.js &\n\
API_PID=$!\n\
echo "--> Waiting 15 seconds for API to boot..."\n\
sleep 15\n\
echo "--> Installing Languages..."\n\
cd /piston_api/cli\n\
node index.js ppman install node python\n\
echo "--> System ready on port 2000."\n\
wait $API_PID\n\
' > /start.sh && chmod +x /start.sh

WORKDIR /piston_api
CMD ["/bin/bash", "/start.sh"]
EXPOSE 2000