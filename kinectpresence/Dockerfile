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

# [NUOVO STEP 4/10] **CRITICO** Configura il percorso di ricerca delle librerie dinamiche per Alpine.
# Questo garantisce che il sistema operativo trovi libfreenect2.so senza LD_LIBRARY_PATH.
RUN mkdir -p /etc/ld.so.conf.d && \
    echo "/usr/local/lib" > /etc/ld.so.conf.d/libfreenect2.conf && \
    ldconfig

WORKDIR /app

# [Step 5/10] Copia lo script Python
COPY kinect_mqtt_bridge.py /app/

# [Step 6/10] Installa le dipendenze di base per la compilazione Python
RUN pip3 install --no-cache-dir --break-system-packages numpy cython setuptools paho-mqtt

# [Step 7/10] Installa pylibfreenect2 dalla sorgente (il linking C++ è ora globale)
RUN git clone https://github.com/r9y9/pylibfreenect2.git /tmp/pylibfreenect2 && \
    cd /tmp/pylibfreenect2 && \
    PKG_CONFIG_PATH="/usr/local/lib/pkgconfig" \
    pip3 install --no-cache-dir --break-system-packages --no-build-isolation . && \
    cd / && \
    rm -rf /tmp/pylibfreenect2

# CONFIGURAZIONE AVVIO CON SCRIPT SEMPLIFICATA
# [Step 8/10] Crea e rende eseguibile lo script di avvio. Rimosse le variabili di percorso.
RUN printf '#!/usr/bin/env bash\n\nset -e\n\n# Messaggio diagnostico di avvio\necho "Inizializzazione script di avvio del container..." 2>&1\n/usr/bin/python3 --version 2>&1\n\n# Esegue lo script Python, reindirizzando stderr (errori) a stdout (log)\nexec /usr/bin/python3 /app/kinect_mqtt_bridge.py 2>&1\n' > /run.sh
RUN chmod +x /run.sh

# [Step 9/10] L'istruzione CMD esegue lo script di avvio.
CMD [ "/run.sh" ]
