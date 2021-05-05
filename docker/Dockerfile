#FROM ubuntu:18.04
ARG IPY_VERSION=2020.2
FROM intelpython/intelpython3_core:${IPY_VERSION}
#FROM python:3.8.5-slim

# Just one extra conda variable
ENV CONDA_PREFIX="/opt/conda"

# Install package dependencies
#RUN apt-get update && \
#    apt-get -y install software-properties-common && \
#    add-apt-repository -y universe && \
#    add-apt-repository -y multiverse && \
#    apt-get -y install \
#      && add-apt-repository -y xenial \
#      && apt-get -y --no-install-recommends install \
#RUN apt-get -y --no-install-recommends install gnupg
#RUN cat /etc/apt/sources.list
#RUN echo 'deb http://dk.archive.ubuntu.com/ubuntu/ xenial main' \
#         'deb http://dk.archive.ubuntu.com/ubuntu/ xenial universe' >> /etc/apt/sources.list
#RUN cat /etc/apt/sources.list
#RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 40976EAF437D05B5 3B4FE6ACC0B21F32
RUN apt-get update -qq \
      && apt-get -y --no-install-recommends install \
         software-properties-common \
         build-essential \
         wget \
         gfortran \
         libglib2.0-dev \
         libpng-dev \
         autoconf \
         autotools-dev \
         automake \
         autogen \
         libtool \
         pkg-config \
         cmake \
         csh \
         gcc \
         git \
         cvs \
         libcfitsio-dev \
         #pgplot \
         hwloc \
         libfftw3-3 \
         libfftw3-bin \
         libfftw3-dev \
         libfftw3-single3 \
         libx11-dev \
         libpnglite-dev \
         libhdf5-dev \
         libhdf5-serial-dev \
         libxml2 \
         libxml2-dev \
         libltdl-dev \
         gsl-bin \
         xauth \
      && apt-get clean all \
      && rm -r /var/lib/apt/lists/*

# Install pgplot
ENV PGPLOT_DIR=/usr/local/pgplot
ENV PGPLOT_DEV=/Xserve
WORKDIR /tmp/pgplot-build
RUN wget ftp://ftp.astro.caltech.edu/pub/pgplot/pgplot5.2.tar.gz \
      && tar zxvf pgplot5.2.tar.gz
WORKDIR /usr/local/pgplot
ADD docker/drivers.list /usr/local/pgplot
RUN /tmp/pgplot-build/pgplot/makemake /tmp/pgplot-build/pgplot linux g77_gcc_aout \
      && sed -i 's/FCOMPL=g77/FCOMPL=gfortran/' makefile \
      && sed -i 's/FFLAGC=-Wall -O/FFLAGC=-shared -Wall -O/' makefile \
      && sed -i 's/FFLAGD=-fno-backslash/FFLAGD=-shared -fno-backslash/' makefile \
      && sed -i 's/CFLAGC=-DPG_PPU -O2 -I./CFLAGC=-fPIC -DPG_PPU -O2 -I./' makefile \
      && sed -i 's/CFLAGD=-O2/CFLAGD=-fPIC -O2/' makefile \
      && make SYSTEM=gfortran STATICLIBS=${PGPLOT_DIR}/libpgplot.a PGPLOTLIBS= \
      && ldconfig \
      && ls *so \
      && make cpg \
      && ldconfig \
      && make clean \
      && rm -r /tmp/pgplot-build


# Install Python packages - conda
ADD docker/requirements.yaml /requirements.yaml
RUN conda install --no-deps -y --file /requirements.yaml \
      && conda clean -ay

# Install presto python scripts
ENV PRESTO /code/presto
ENV LD_LIBRARY_PATH /code/presto/lib
ADD . /code/presto
WORKDIR /code/presto/src
RUN make libpresto slalib
WORKDIR /code/presto
RUN python setup.py install \
#--prefix /code/presto \
      && sed -i 's/env python/env python3/' /code/presto/bin/*py \
      && python3 tests/test_presto_python.py 


# Installs all the C dependancies -----------------------------
WORKDIR /tmp

# Install psrcat
RUN mkdir -p /tmp/psrcat-build \
      && cd /tmp/psrcat-build \
      && wget https://www.atnf.csiro.au/research/pulsar/psrcat/downloads/psrcat_pkg.tar.gz \
      && gunzip psrcat_pkg.tar.gz \
      && tar -xvf psrcat_pkg.tar \
    #rm psrcat_pkg.tar && \
      && cd psrcat_tar \
      && bash makeit \
      && ldconfig \
      && cp psrcat /usr/bin \
      && cd / \
      && rm -rf /tmp/psrcat-build

    
    
# Install tempo
RUN git clone https://github.com/nanograv/tempo.git && \
    cd tempo && \
    ./prepare && \
    ./configure && \
    make && \
    make install
ENV TEMPO /home/soft/tempo

# Install tempo2
WORKDIR /home/soft/tempo2
ENV TEMPO2 /home/soft/tempo2
RUN wget https://sourceforge.net/projects/tempo2/files/latest/download && \
    tar -xzvf download && \
    rm download && \
    cp -r tempo2*/T2runtime/ $TEMPO2
# Install presto
WORKDIR /code/presto/src
RUN make makewisdom && \
    make prep && \
    make -j 1 && \
    make clean
ENV PATH="/code/presto/bin/:${PATH}"

