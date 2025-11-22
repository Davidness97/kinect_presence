ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base:latest 
FROM $BUILD_FROM

# 1. Installazione delle dipendenze di sistema (necessarie per la compilazione e runtime)
RUN apk update \
    && apk add --no-cache \
        # Dipendenze per LibFreenect2 e Python
        build-base \
        cmake \
        libusb-dev \
        mesa-dev \
        pkgconfig \
        python3-dev \
        py3-pip \
        # Tool di basso livello per USB e file
        jq \
        git \
    && rm -rf /var/cache/apk/*

# 2. Definizione dell'ambiente Python e installazione di Mqtt
WORKDIR /app
COPY kinect_mqtt_bridge.py /app/

# 1. Installazione di NumPy e dipendenze di build (per risolvere il ModuleNotFoundError)
RUN pip3 install \
    --no-cache-dir \
    --break-system-packages \
    numpy \
    cython \              # <-- AGGIUNGI QUESTO
    setuptools            # <-- AGGIUNGI QUESTO (per setuptools/build_meta.py)

# 2. Installazione delle dipendenze principali (Ora NumPy, Cython e setuptools sono disponibili)
RUN pip3 install \
    --no-cache-dir \
    --break-system-packages \
    paho-mqtt \
    pylibfreenect2

# 3. Copia del run.sh e impostazione dei permessi
# Nota: run.sh viene copiato in /usr/bin e non in /app
COPY run.sh /usr/bin/run.sh
RUN chmod a+x /usr/bin/run.sh

# 4. ENTRYPOINT: L'esecuzione parte da qui
CMD [ "/usr/bin/run.sh" ]







