FROM ubuntu:18.04

ENV DEBIAN_FRONTEND=noninteractive

# Install core development tools, editors, and dependencies
RUN apt-get update && apt-get install -y \
    software-properties-common \
    git curl wget build-essential cmake \
    autoconf libtool pkg-config \
    libfftw3-dev libboost-all-dev \
    libgmp-dev swig python3 python3-dev python3-pip \
    gnuradio vim nano && \
    rm -rf /var/lib/apt/lists/*

# Install kernel module tools and extra runtime dependencies
RUN apt-get update && apt-get install -y \
    kmod nano libcap2-bin libgnuradio-osmosdr0.1.4

# Add Ettus Research PPA and install UHD packages
RUN add-apt-repository -y ppa:ettusresearch/uhd && \
    apt-get update && \
    apt-get install -y libuhd-dev libuhd003 uhd-host && \
    rm -rf /var/lib/apt/lists/*

# Download and extract the OpenLTE tarball
WORKDIR /opt
RUN wget https://sourceforge.net/projects/openlte/files/openlte_v00-20-04.tgz -O openlte.tar.gz && \
    tar -xvzf openlte.tar.gz && \
    rm openlte.tar.gz

# Set working directory to OpenLTE source and configure git
WORKDIR /opt/openlte_v00-20-04
RUN git config --global --add safe.directory /opt/openlte_v00-20-04 && \
    git config --global user.name "FrosT2k5" && \
    git config --global user.email "yashpatil2k5@gmail.com"

# Download and apply the build fix patch
RUN wget https://raw.githubusercontent.com/FirmWire/ndss22_experiments/refs/heads/main/V-G_ota_reproduction/rrc1/openlte_buildfix.patch && \
    git apply openlte_buildfix.patch && \
    rm openlte_buildfix.patch

RUN apt-get update && apt-get install -y libbladerf-dev libmbedtls-dev iptables net-tools

# Build openlte
RUN mkdir build && cd build && \
    cmake .. && \
    make -j12

# Set working directory to build folder for runtime usage
WORKDIR /opt/openlte_v00-20-04/build

# Default to interactive shell inside the build directory
CMD ["/bin/bash"]

# Run the container using command: 
# docker run -it --privileged --device /dev/bus/usb -p 30000:30000 -p 30001:30001 -p 30002:30002 -p 30003:30003 -v /lib/modules:/lib/modules -v /etc/network/if-up.d:/etc/network/if-up.d openlte
# run make install to install openlte
# command to build: docker build -t openlte .
