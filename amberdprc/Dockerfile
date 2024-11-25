FROM almalinux:8

ARG CUDA_VERSION=126
ARG XTB_VERSION=6.7.0
ARG DEEPMD_VERSION=3.0.0
ARG AMBER_VERSION=24_rc5

ENV DEEPMDHOME="/deepmd-kit"
ENV AMBERHOME="/amber"

COPY --from=ghcr.io/deepmodeling/deepmd-kit:${DEEPMD_VERSION}_cuda${CUDA_VERSION} /opt/deepmd-kit ${DEEPMDHOME}

RUN dnf --enablerepo=powertools -y install tcsh make \
        gcc gcc-gfortran gcc-c++ \
        which flex bison patch bc \
        libXt-devel libXext-devel \
        perl perl-ExtUtils-MakeMaker util-linux wget \
        bzip2 bzip2-devel zlib-devel tar \
        openmpi-devel cmake git \
        openblas-devel

# build xtb
RUN wget https://codeload.github.com/grimme-lab/xtb/zip/v${XTB_VERSION} -O xtb.zip \
    && unzip xtb.zip \
    && mkdir xtb-${XTB_VERSION}/build \
    && cd xtb-${XTB_VERSION}/build \
    && cmake .. -DCMAKE_INSTALL_PREFIX=${AMBERHOME} \
                -DINSTALL_MODULES=ON \
    && cmake --build . --target install \
    && cd ../.. \
    && rm -rf xtb.zip xtb-${XTB_VERSION}

# compile AMBER
RUN wget https://ambermd.org/downloads/AmberTools${AMBER_VERSION}.tar.bz2 -O AmberTools.tar.bz2 \
    && tar -xjf AmberTools.tar.bz2 \
    && rm AmberTools.tar.bz2 \
    && cd amber${AMBER_VERSION:0:2}_src/build \
    && cmake .. \
             -DCMAKE_INSTALL_PREFIX=${AMBERHOME} \
             -DCMAKE_PREFIX_PATH="/usr/lib64/openmpi;${AMBERHOME};${DEEPMD_VERSION}" \
             -DCOMPILER=GNU \
             -DMPI=ON \
             -DCUDA=OFF \
             -DDOWNLOAD_MINICONDA=OFF \
             -DBUILD_PYTHON=OFF \
             -DUSE_XTB=ON \
             -DUSE_DEEPMDKIT=ON \
    && cmake --build . --target install \
    && cd ../.. \
    && rm -rf amber${AMBER_VERSION:0:2}_src

ENV PATH="${AMBERHOME}/bin:/usr/lib64/openmpi/bin:${PATH}"
ENV LD_LIBRARY_PATH="${AMBERHOME}/lib:${AMBERHOME}/lib64:${LD_LIBRARY_PATH}"
ENV PERL5LIB="${AMBERHOME}/lib/perl:${PERL5LIB}"
ENV QUICK_BASIS="${AMBERHOME}/AmberTools/src/quick/basis"
ENV OMPI_ALLOW_RUN_AS_ROOT=1
ENV OMPI_ALLOW_RUN_AS_ROOT_CONFIRM=1

LABEL org.opencontainers.image.source=https://github.com/njzjz/docker-amberdprc
LABEL org.opencontainers.image.title="Amber DPRc"
LABEL org.opencontainers.image.description="Amber DPRc"
LABEL org.opencontainers.image.licenses=LGPL-3.0-or-later
LABEL org.opencontainers.image.authors="Jinzhe Zeng <jinzhe.zeng@rutgers.edu>"
LABEL org.opencontainers.image.url=https://github.com/njzjz/docker-amberdprc
LABEL org.opencontainers.image.documentation=https://github.com/njzjz/docker-amberdprc
LABEL org.opencontainers.image.vendor="Jinzhe Zeng"

CMD ["/bin/bash"]
