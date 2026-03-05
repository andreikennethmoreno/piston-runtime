FROM buildpack-deps:buster AS isolate
RUN apt-get update && \
    apt-get install -y --no-install-recommends git libcap-dev && \
    rm -rf /var/lib/apt/lists/* && \
    git clone https://github.com/envicutor/isolate.git /tmp/isolate/ && \
    cd /tmp/isolate && \
    git checkout af6db68042c3aa0ded80787fbb78bc0846ea2114 && \
    make -j$(nproc) install && \
    rm -rf /tmp/*

FROM node:15.10.0-buster-slim

ENV DEBIAN_FRONTEND=noninteractive

RUN dpkg-reconfigure -p critical dash
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

# Set working directory
WORKDIR /piston_api

# 🟢 FIX: Copy both API and CLI folders so ppman works internally
COPY ./api/package.json ./api/package-lock.json ./
RUN npm install
COPY ./api/src ./src
COPY ./cli ./cli

# 🟢 FIX: Install CLI dependencies so we can run ppman
RUN cd cli && npm install

# ==============================================================================
# BAKE LANGUAGES INTO THE IMAGE
# ==============================================================================
RUN node cli/index.js ppman install node=18.15.0 && \
    node cli/index.js ppman install typescript=5.0.3 && \
    node cli/index.js ppman install python=3.10.0 && \
    node cli/index.js ppman install php=8.2.3 && \
    node cli/index.js ppman install ruby=3.0.1 && \
    node cli/index.js ppman install gcc=10.2.0 && \
    node cli/index.js ppman install go=1.16.2 && \
    node cli/index.js ppman install rust=1.68.2 && \
    node cli/index.js ppman install mono=6.12.0 && \
    node cli/index.js ppman install swift=5.3.3 && \
    node cli/index.js ppman install java=15.0.2 && \
    node cli/index.js ppman install kotlin=1.8.20

CMD ["/piston_api/src/docker-entrypoint.sh"]
EXPOSE 2000/tcp