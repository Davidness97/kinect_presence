FROM ghcr.io/home-assistant/amd64-base:latest

# [Step 2/10] Installa le dipendenze di build e di sistema
RUN apk update && \
    apk add --no-cache build-base cmake libusb-dev mesa-dev pkgconfig python3-dev py3-pip jq git libjpeg-turbo-dev libx11-dev && \
    rm -rf /var/cache/apk/*

# [Step 3/10] Clona, compila e installa libfreenect2
# Questo crea la libreria C++ in /usr/local/
RUN git clone https://github.com/OpenKinect/libfreenect2.git /tmp/libfreenect2 && \
    mkdir -p /tmp/libfreenect2/build && \
    cd /tmp/libfreenect2/build && \
    cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local && \
    make -j$(nproc) && \
    make install && \
    cd / && \
    rm -rf /tmp/libfreenect2

WORKDIR /app

# [Step 6/10] Copia lo script Python
COPY kinect_mqtt_bridge.py /app/

# [Step 7/10] Installa le dipendenze di base per la compilazione Python
RUN pip3 install --no-cache-dir --break-system-packages numpy cython setuptools paho-mqtt

# [Step 8/10 - CORRETTO] Installa pylibfreenect2 dalla sorgente per trovare i file Cython .pxd
RUN git clone https://github.com/r9y9/pylibfreenect2.git /tmp/pylibfreenect2 && \
    cd /tmp/pylibfreenect2 && \
    PKG_CONFIG_PATH="/usr/local/lib/pkgconfig" \
    LD_LIBRARY_PATH="/usr/local/lib" \
    pip3 install --no-cache-dir --break-system-packages --no-build-isolation . && \
    cd / && \
    rm -rf /tmp/pylibfreenect2

# NUOVI STEP FONDAMENTALI PER S6-OVERLAY (Home Assistant Add-on)
# Definisce il tuo script Python come un servizio gestito da S6.

# 1. Crea la directory per il servizio S6 (lo chiamo 'kinect-service')
RUN mkdir -p /etc/s6-overlay/s6-rc.d/kinect-service
# 2. Crea lo script 'run' eseguibile usando 'exec' (necessario per S6)
RUN printf '#!/usr/bin/env bash\n\nexec python3 /app/kinect_mqtt_bridge.py\n' > /etc/s6-overlay/s6-rc.d/kinect-service/run
# 3. Rende lo script eseguibile
RUN chmod +x /etc/s6-overlay/s6-rc.d/kinect-service/run

# 4. Crea la directory contents.d che la base image non crea automaticamente
RUN mkdir -p /etc/s6-overlay/s6-rc.d/default/contents.d

# 5. Aggiunge il servizio al bundle 'default' per l'avvio automatico
RUN printf 'kinect-service\n' > /etc/s6-overlay/s6-rc.d/default/contents.d/kinect-service
