# Copyright 2018 the u-root Authors. All rights reserved
# Use of this source code is governed by a BSD-style
# license that can be found in the LICENSE file.

# Build:
#
#     sudo docker build . -t uroottest/x86-integration
#
# Run:
#
#     sudo docker run --rm -it uroottest/x86-integration
#
# More instructions:
#
#     https://circleci.com/docs/2.0/custom-images/

FROM circleci/golang:1.11

# Install dependencies
RUN sudo apt-get update &&                          \
    sudo apt-get install -y --no-install-recommends \
        `# Linux dependencies`                      \
        git                                         \
        bc                                          \
        bison                                       \
        flex                                        \
        gcc                                         \
        make                                        \
        `# QEMU dependencies`                       \
        libglib2.0-dev                              \
        libfdt-dev                                  \
        libpixman-1-dev                             \
        zlib1g-dev &&                               \
    sudo rm -rf /var/lib/apt/lists/*

# Create working directory
WORKDIR /home/circleci
COPY config_linux4.17_amd64.txt .config

# Build linux
RUN set -eux;                                                             \
    git clone --depth=1 --branch=v4.17 https://github.com/torvalds/linux; \
    mv .config linux/;                                                    \
    cd linux;                                                             \
    make -j$(($(nproc) * 2 + 1));                                         \
    cd ~;                                                                 \
    cp linux/arch/x86_64/boot/bzImage bzImage;                            \
    rm -rf linux/

# Build QEMU
RUN set -eux;                                                          \
    git clone --depth=1 --branch=v2.12.0 https://github.com/qemu/qemu; \
    cd qemu;                                                           \
    mkdir build;                                                       \
    cd build;                                                          \
    ../configure                                                       \
        --target-list=x86_64-softmmu                                   \
        --disable-docs                                                 \
        --disable-sdl                                                  \
        --disable-kvm;                                                 \
    make -j$(($(nproc) * 2 + 1));                                      \
    cd ~;                                                              \
    cp -rL qemu/build/pc-bios/ ~/pc-bios;                              \
    cp qemu/build/x86_64-softmmu/qemu-system-x86_64 .;                 \
    rm -rf qemu/

# Export paths to binaries.
ENV UROOT_KERNEL /home/circleci/bzImage
ENV UROOT_QEMU "/home/circleci/qemu-system-x86_64 -L /home/circleci/pc-bios"
