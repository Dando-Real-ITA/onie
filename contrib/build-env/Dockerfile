#  Copyright (C) 2018 Curt Brune <curt@cumulusnetworks.com>
#
#  Modified with permission from Benayak Karki
#
#  SPDX-License-Identifier:     GPL-2.0

FROM debian:10

# Configure Debian archive repositories for end-of-life release
RUN cat > /etc/apt/sources.list << EOF
deb http://archive.debian.org/debian/ buster main contrib non-free
deb http://archive.debian.org/debian-security/ buster/updates main contrib non-free
EOF

# Configure apt to handle old timestamps and unsigned packages
RUN echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99archive && \
    echo 'Acquire::AllowInsecureRepositories "true";' >> /etc/apt/apt.conf.d/99archive && \
    echo 'Acquire::AllowDowngradeToInsecureRepositories "true";' >> /etc/apt/apt.conf.d/99archive

RUN apt-get update

# Add initial development packages
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y --allow-unauthenticated --no-install-recommends \
    build-essential stgit u-boot-tools util-linux \
    gperf device-tree-compiler python-all-dev xorriso \
    autoconf automake bison flex texinfo libtool libtool-bin \
    realpath gawk libncurses5 libncurses5-dev bc \
    dosfstools mtools pkg-config git wget help2man libexpat1 \
    libexpat1-dev fakeroot python-sphinx rst2pdf equivs \
    libefivar-dev libnss3-tools libnss3-dev libpopt-dev openssl x11-xkb-utils \
    libssl-dev sbsigntool uuid-runtime uuid-dev cpio nano gnupg2 rsync libfreetype6-dev \
    bsdmainutils curl sudo unzip tree python3.7-dev python3-distutils ca-certificates locales-all \
    libelf-dev ispell jq qemu-utils ovmf gnu-efi libfile-slurp-perl grub-common psmisc monkeysphere

# Create build user
RUN useradd -m -s /bin/bash build && \
        adduser build sudo && \
        echo "build:build" | chpasswd

WORKDIR /home/build

# Add /sbin and /usr/sbin to build user's path
RUN echo export PATH="/sbin:/usr/sbin:\$PATH" >> .bashrc

# Add common files, like .gitconfig
COPY home .

# Create /home/build/src as a mount point for sharing files with the
# host system.
RUN mkdir src

# Make sure everything in /home/build is owned by the build user
RUN chown -R build:build .

USER build

CMD ["/bin/bash", "--login"]
