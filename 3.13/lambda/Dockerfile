ARG PROJ_VERSION=9.5.1
ARG GEOS_VERSION=3.11.5
ARG GDAL_VERSION=3.10.0

FROM public.ecr.aws/lambda/python:3.13 AS builder

ARG PROJ_VERSION
ARG GEOS_VERSION
ARG GDAL_VERSION

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
RUN wget https://download.osgeo.org/proj/proj-${PROJ_VERSION}.tar.gz \
    && tar -xzf proj-${PROJ_VERSION}.tar.gz \
    && cd proj-${PROJ_VERSION} \
    && mkdir build && cd build \
    && cmake -DCMAKE_INSTALL_PREFIX=${INSTALL_PREFIX} .. \
    && cmake --build . -j$(nproc) \
    && make install

### Build GEOS
WORKDIR /build/geos
RUN wget https://download.osgeo.org/geos/geos-${GEOS_VERSION}.tar.bz2 \
    && tar -xjf geos-${GEOS_VERSION}.tar.bz2 \
    && cd geos-${GEOS_VERSION} \
    && mkdir build && cd build \
    && cmake -DCMAKE_INSTALL_PREFIX=${INSTALL_PREFIX} .. \
    && cmake --build . -j$(nproc) \
    && make install

### Build GDAL
WORKDIR /build/gdal
RUN wget https://github.com/OSGeo/gdal/releases/download/v${GDAL_VERSION}/gdal-${GDAL_VERSION}.tar.gz \
    && tar -xzf gdal-${GDAL_VERSION}.tar.gz \
    && cd gdal-${GDAL_VERSION} \
    && mkdir build && cd build \
    && cmake -DCMAKE_INSTALL_PREFIX=${INSTALL_PREFIX} .. \
    && cmake --build . -j$(nproc) \
    && make install

FROM public.ecr.aws/lambda/python:3.13

ARG PROJ_VERSION
ARG GEOS_VERSION
ARG GDAL_VERSION

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

ENV LD_LIBRARY_PATH="/usr/local/lib64:${LD_LIBRARY_PATH:-}" \
    GDAL_LIBRARY_PATH=/usr/local/lib64/libgdal.so \
    GEOS_LIBRARY_PATH=/usr/local/lib64/libgeos_c.so \
    PROJ_VERSION=${PROJ_VERSION} \
    GEOS_VERSION=${GEOS_VERSION} \
    GDAL_VERSION=${GDAL_VERSION}

LABEL \
  gdal.version="${GDAL_VERSION}" \
  geos.version="${GEOS_VERSION}" \
  proj.version="${PROJ_VERSION}"

# Test versions of installed libraries (verifies correct installation)
RUN set -ex  \
    && test $(proj 2>&1 | head -n 1 | grep -oP 'Rel\.\s+\K[0-9.]+') = "${PROJ_VERSION}" \
    && test $(geos-config --version) = "${GEOS_VERSION}" \
    && test $(gdalinfo --version | awk '{sub(/,/, "", $2); print $2}') = "${GDAL_VERSION}"