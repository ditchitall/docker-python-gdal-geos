FROM python:3.12-slim-bookworm

ARG GDAL_VERSION=3.10.0
ARG SOURCE_DIR=/usr/local/src/python-gdal
ARG GDAL_PYTHON_BINDINGS_WITHOUT_NUMPY=1
ARG GDAL_INSTALL_PREFIX=/opt/gdal

ENV GDAL_PYTHON_BINDINGS_WITHOUT_NUMPY=${GDAL_PYTHON_BINDINGS_WITHOUT_NUMPY} \
    LD_LIBRARY_PATH="${GDAL_INSTALL_PREFIX}/lib:${LD_LIBRARY_PATH}"

RUN \
    # Install runtime dependencies
    apt-get update \
    && BUILD_PKGS="build-essential wget git-core libsqlite3-dev xz-utils libxml2-dev liblzma-dev curl cmake libproj-dev" \
    && RUNTIME_PKGS="ca-certificates swig libgeos-dev libxml2 libdeflate0 libtiff6 libproj25" \
    && apt-get install -y --no-install-recommends ${BUILD_PKGS} ${RUNTIME_PKGS} \
    && pip install --upgrade pip setuptools wheel --no-cache-dir \
    && rm -rf /var/lib/apt/lists/* \
    # Install GDAL
    && export CMAKE_BUILD_PARALLEL_LEVEL=`nproc --all` \
    && mkdir -p "${SOURCE_DIR}" \
    && cd "${SOURCE_DIR}" \
    && wget "http://download.osgeo.org/gdal/${GDAL_VERSION}/gdal-${GDAL_VERSION}.tar.gz" \
    && tar -xvf "gdal-${GDAL_VERSION}.tar.gz" \
    && cd gdal-${GDAL_VERSION} \
    && mkdir build \
    && cd build \
    && cmake .. \
        -DBUILD_PYTHON_BINDINGS=ON \
        -DCMAKE_BUILD_TYPE=Release \
        -DPYTHON_INCLUDE_DIR=`python -c "import sysconfig; print(sysconfig.get_path('include'))"` \
        -DPYTHON_LIBRARY=`python -c "import sysconfig; print(sysconfig.get_config_var('LIBDIR'))"`\
        -DCMAKE_INSTALL_PREFIX=${GDAL_INSTALL_PREFIX} \
        -DGDAL_PYTHON_INSTALL_PREFIX=${GDAL_INSTALL_PREFIX} \
    && cmake --build . \
    && cmake --build . --target install \
    && ldconfig \
    # Clean-up
    && apt-get update -y \
    && apt-get remove -y --purge ${BUILD_PKGS} \
    && apt-get autoremove -y \
    && apt-get install -y --no-install-recommends ${RUNTIME_PKGS} \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf "${SOURCE_DIR}"

ENV PATH="${GDAL_INSTALL_PREFIX}/bin:$PATH" \
    PKG_CONFIG_PATH="${GDAL_INSTALL_PREFIX}/lib/pkgconfig:${PKG_CONFIG_PATH}" \
    PYTHONPATH="${GDAL_INSTALL_PREFIX}/lib/python3.12/site-packages:$PYTHONPATH" \
    GDAL_LIBRARY_PATH="${GDAL_INSTALL_PREFIX}/lib/libgdal.so"

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    gnupg \
    apt-transport-https \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -m 0755 -p /etc/apt/keyrings && \
    curl -fsSL https://ftp-master.debian.org/keys/archive-key-12.asc | gpg --dearmor -o /etc/apt/keyrings/debian-archive-keyring.gpg

#RUN python -c "from osgeo import gdal; print(gdal.__file__)"

CMD ["python", "-V", "&&", "pip", "-V", "&&", "gdalinfo", "--version"]
