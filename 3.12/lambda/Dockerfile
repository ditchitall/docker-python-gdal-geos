FROM public.ecr.aws/lambda/python:3.12

# Install build dependencies
RUN dnf install -y \
  cmake \
  g++ \
  gcc \
  libcurl-devel \
  libtiff-devel \
  sqlite-devel \
  tar \
  wget \
  zlib-devel
RUN dnf clean all

# Install Proj
# https://proj.org/en/stable/install.html#compilation-and-installation-from-source-code
WORKDIR /proj
RUN wget https://download.osgeo.org/proj/proj-9.5.1.tar.gz \
    && tar -xzf proj-9.5.1.tar.gz \
    && cd proj-9.5.1 \
    && mkdir build && cd build \
    && cmake .. && cmake --build . -j$(nproc) \
    && cmake -DCMAKE_INSTALL_PREFIX=/usr/local .. \
    && make install -j$(nproc)
RUN rm -rf /proj/proj-9.5.1*

# Install GDAL
# https://gdal.org/en/stable/development/building_from_source.html
WORKDIR /gdal
RUN wget https://github.com/OSGeo/gdal/releases/download/v3.10.0/gdal-3.10.0.tar.gz \
    && tar -xzf gdal-3.10.0.tar.gz \
    && cd gdal-3.10.0 \
    && mkdir build && cd build \
    && cmake .. && cmake --build . -j$(nproc) \
    && cmake -DCMAKE_INSTALL_PREFIX=/usr/local .. \
    && make install -j$(nproc)
RUN rm -rf /gdal/gdal-3.10.0*

WORKDIR /geos
RUN wget https://download.osgeo.org/geos/geos-3.11.5.tar.bz2 \
    && dnf install -y bzip2 \
    && tar -xjf geos-3.11.5.tar.bz2 \
    && cd geos-3.11.5 \
    && mkdir build && cd build \
    && cmake .. && cmake --build . -j$(nproc) \
    && cmake -DCMAKE_INSTALL_PREFIX=/usr/local .. \
    && make install -j$(nproc) \
    && dnf clean all
RUN rm -rf /geos/geos-3.11.5*

ENV LD_LIBRARY_PATH="/usr/local/lib64:$LD_LIBRARY_PATH" \
    GDAL_LIBRARY_PATH=/usr/local/lib64/libgdal.so \
    GEOS_LIBRARY_PATH=/usr/local/lib64/libgeos_c.so

ENV GEOIP2FAST_PATH=/opt/ldr/geoip2fast
RUN mkdir -p ${GEOIP2FAST_PATH} \
    && curl -L -o "${GEOIP2FAST_PATH}/geoip2fast.dat.gz" https://github.com/rabuchaim/geoip2fast/releases/latest/download/geoip2fast.dat.gz

# Switch back to default
WORKDIR /var/task
