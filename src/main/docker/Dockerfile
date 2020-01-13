#
# Copyright 2019-2020 Micro Focus or one of its affiliates.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

FROM cafapi/opensuse-base:1

ENV PYTHON_VERSION 3.7.6
ENV GPG_KEY 0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D

ARG DEVEL_DEPS="\
    gcc \
    make \
    wget \
    tar \
    automake \
    pkgconfig \
    libbz2-devel \
    bluez-devel \
    libexpat-devel \
    libffi-devel \
    lzma-devel \
    openssl-devel \
    libuuid-devel \
    zlib-devel \
    fdupes \
    netcfg \
    timezone \
    xz \
    "

# Build Python from source
RUN zypper -n refresh && \
    zypper -n in $DEVEL_DEPS && \
    wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" && \
    wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" && \
    \
    export GNUPGHOME="$(mktemp -d)" && \
# https://github.com/docker-library/tomcat/issues/87
    for server in ha.pool.sks-keyservers.net \
                  pgp.mit.edu \
                  hkp://p80.pool.sks-keyservers.net:80 \
                  keyserver.ubuntu.com \
                  hkp://keyserver.ubuntu.com:80 \
                  keyserver.pgp.com; do \
        ( gpg --batch --keyserver "$server" --recv-keys "$GPG_KEY" && break || echo "Trying new server..." ) \
    done && \
    gpg --batch --verify python.tar.xz.asc python.tar.xz && \
    { command -v gpgconf > /dev/null && gpgconf --kill all || :; } && \
    rm -rf "$GNUPGHOME" python.tar.xz.asc && \
    \
    mkdir -p /usr/src/python && \
    tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz && \
    rm python.tar.xz && \
    \
    cd /usr/src/python && \
    ./configure \
        --enable-loadable-sqlite-extensions \
        --enable-optimizations \
        --enable-option-checking=fatal \
        --enable-shared \
        --with-system-expat \
        --with-system-ffi \
        --without-ensurepip && \
    make -j8 \
# https://github.com/docker-library/python/issues/160#issuecomment-509426916
        PROFILE_TASK='-m test.regrtest --pgo \
            test_array \
            test_base64 \
            test_binascii \
            test_binhex \
            test_binop \
            test_bytes \
            test_c_locale_coercion \
            test_class \
            test_cmath \
            test_codecs \
            test_compile \
            test_complex \
            test_csv \
            test_decimal \
            test_dict \
            test_float \
            test_fstring \
            test_hashlib \
            test_io \
            test_iter \
            test_json \
            test_long \
            test_math \
            test_memoryview \
            test_pickle \
            test_re \
            test_set \
            test_slice \
            test_struct \
            test_threading \
            test_time \
            test_traceback \
            test_unicode \
        ' && \
    make install && \
    ldconfig && \
    \
    cd / && \
    rm -rf /usr/src/python && \
    find /usr/local -depth \
        \( \
            \( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
            -o \
            \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
        \) -exec rm -rf '{}' + && \
    python3 --version && \
    zypper -n rm --clean-deps $DEVEL_DEPS && \
    zypper clean --all

# Some useful symlinks
RUN cd /usr/local/bin && \
    ln -s idle3 idle && \
    ln -s pydoc3 pydoc && \
    ln -s python3 python && \
    ln -s python3-config python-config

ENV PYTHON_PIP_VERSION 19.3.1
ENV PYTHON_GET_PIP_URL https://github.com/pypa/get-pip/raw/ffe826207a010164265d9cc807978e3604d18ca0/get-pip.py
ENV PYTHON_GET_PIP_SHA256 b86f36cc4345ae87bfd4f10ef6b2dbfa7a872fbff70608a1e43944d283fd0eee

# Install pip
RUN zypper -n in wget && \
    wget -O get-pip.py "$PYTHON_GET_PIP_URL" && \
    echo "$PYTHON_GET_PIP_SHA256 *get-pip.py" | sha256sum --check --strict - && \
    \
    python get-pip.py \
        --disable-pip-version-check \
        --no-cache-dir \
        "pip==$PYTHON_PIP_VERSION" && \
    pip --version && \
    \
    rm -f get-pip.py && \
    zypper -n rm --clean-deps wget && \
    zypper clean --all

# Tag the image
ARG BUILD_NUMBER
ARG BUILD_DATE
ARG GIT_REPO
ARG GIT_BRANCH
ARG GIT_COMMIT

LABEL Build.Number="$BUILD_NUMBER" \
      Build.Date="$BUILD_DATE" \
      Git.Repo="$GIT_REPO" \
      Git.Branch="$GIT_BRANCH" \
      Git.Commit="$GIT_COMMIT"

CMD ["python3"]
