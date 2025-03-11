FROM linuxmintd/mint20.3-amd64

# Etkileşimsiz kurulum için
ENV DEBIAN_FRONTEND=noninteractive

# Temel paketleri güncelleme ve yükleme
RUN apt-get update && apt-get install -y --no-install-recommends \
    apt-utils \
    curl \
    ca-certificates \
    software-properties-common \
    gnupg2 \
    lsb-release \
    locales \
    git \
    openssh-client \
    sudo \
    wget \
    vim \
    zsh \
    build-essential \
    python3-pip \
    python3-dev \
    systemd \
    systemd-sysv \
    dbus \
    dbus-x11 \
    tigervnc-standalone-server \
    tigervnc-common \
    websockify \
    supervisor \
    net-tools \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Locale'leri ayarlama
RUN localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Linux Mint masaüstü ortamını yükleme (Cinnamon varsayılan olarak gelir)
RUN apt-get update && apt-get install -y --no-install-recommends \
    mint-meta-cinnamon \
    firefox \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Docker kurulumu
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - \
    && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" \
    && apt-get update \
    && apt-get install -y docker-ce docker-ce-cli containerd.io \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# noVNC'yi GitHub'dan klonla
RUN git clone https://github.com/novnc/noVNC.git /opt/novnc \
    && cd /opt/novnc \
    && git checkout v1.4.0 \
    && ln -s /opt/novnc/vnc.html /opt/novnc/index.html

# systemd için düzgün hazırlık
RUN cd /lib/systemd/system/sysinit.target.wants/ && \
    ls | grep -v systemd-tmpfiles-setup | xargs rm -f && \
    rm -f /lib/systemd/system/multi-user.target.wants/* && \
    rm -f /etc/systemd/system/*.wants/* && \
    rm -f /lib/systemd/system/local-fs.target.wants/* && \
    rm -f /lib/systemd/system/sockets.target.wants/*udev* && \
    rm -f /lib/systemd/system/sockets.target.wants/*initctl* && \
    rm -f /lib/systemd/system/basic.target.wants/* && \
    mkdir -p /run/systemd && \
    echo "docker" > /run/systemd/container

# systemd'nin gereksiz servislerini devre dışı bırakma
RUN systemctl mask \
    dev-hugepages.mount \
    sys-fs-fuse-connections.mount \
    systemd-logind.service \
    getty.target \
    console-getty.service

# vscode kullanıcısını oluşturma
RUN groupadd --gid 1000 vscode \
    && useradd --uid 1000 --gid 1000 -m vscode \
    && echo vscode ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/vscode \
    && chmod 0440 /etc/sudoers.d/vscode

# VNC sunucusu için parola ayarlama
RUN mkdir -p /home/vscode/.vnc && \
    echo "password" | vncpasswd -f > /home/vscode/.vnc/passwd && \
    chmod 600 /home/vscode/.vnc/passwd && \
    chown -R vscode:vscode /home/vscode/.vnc

# VNC için systemd servis dosyası
COPY vncserver@.service /etc/systemd/system/
RUN chmod 644 /etc/systemd/system/vncserver@.service

# noVNC için yapılandırma
COPY novnc.service /etc/systemd/system/
RUN chmod 644 /etc/systemd/system/novnc.service

# systemd'nin çalışacağını kesin olarak belirt
STOPSIGNAL SIGRTMIN+3

# Giriş komutu - systemd'yi PID 1 olarak başlat
ENTRYPOINT ["/lib/systemd/systemd"]
