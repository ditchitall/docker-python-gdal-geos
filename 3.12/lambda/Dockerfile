FROM public.ecr.aws/lambda/python:3.12 as base

# Final runtime image base
WORKDIR /var/task

ENV LD_LIBRARY_PATH="/usr/local/lib64:$LD_LIBRARY_PATH" \
    GDAL_LIBRARY_PATH=/usr/local/lib64/libgdal.so \
    GEOS_LIBRARY_PATH=/usr/local/lib64/libgeos_c.so \
    GEOIP2FAST_PATH=/opt/ldr/geoip2fast

RUN mkdir -p ${GEOIP2FAST_PATH} \
    && curl -L -o "${GEOIP2FAST_PATH}/geoip2fast.dat.gz" https://github.com/rabuchaim/geoip2fast/releases/latest/download/geoip2fast.dat.gz

# -------------------------------------------------------------------------------------

FROM public.ecr.aws/lambda/python:3.12 as builder

RUN dnf install -y \
  cmake \
  g++ \
  gcc \
  libcurl-devel \
  libtiff-devel \
  sqlite-devel \
  tar \
  wget \
  bzip2 \
  zlib-devel \
  make \
  && dnf clean all

ENV INSTALL_PREFIX=/usr/local

### Build PROJ
WORKDIR /build/proj
RUN wget https://download.osgeo.org/proj/proj-9.5.1.tar.gz \
    && tar -xzf proj-9.5.1.tar.gz \
    && cd proj-9.5.1 \
    && mkdir build && cd build \
    && cmake -DCMAKE_INSTALL_PREFIX=${INSTALL_PREFIX} .. \
    && cmake --build . -j$(nproc) \
    && make install

### Build GEOS
WORKDIR /build/geos
RUN wget https://download.osgeo.org/geos/geos-3.11.5.tar.bz2 \
    && tar -xjf geos-3.11.5.tar.bz2 \
    && cd geos-3.11.5 \
    && mkdir build && cd build \
    && cmake -DCMAKE_INSTALL_PREFIX=${INSTALL_PREFIX} .. \
    && cmake --build . -j$(nproc) \
    && make install

### Build GDAL
WORKDIR /build/gdal
RUN wget https://github.com/OSGeo/gdal/releases/download/v3.10.0/gdal-3.10.0.tar.gz \
    && tar -xzf gdal-3.10.0.tar.gz \
    && cd gdal-3.10.0 \
    && mkdir build && cd build \
    && cmake -DCMAKE_INSTALL_PREFIX=${INSTALL_PREFIX} .. \
    && cmake --build . -j$(nproc) \
    && make install

# -------------------------------------------------------------------------------------

FROM base

RUN dnf install -y \
    libtiff \
    zlib \
    sqlite \
    && dnf clean all

# Copy compiled binaries from builder
COPY --from=builder /usr/local/lib64 /usr/local/lib64
COPY --from=builder /usr/local/include /usr/local/include
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /usr/local/share /usr/local/share