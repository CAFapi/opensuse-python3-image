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

FROM cafapi/opensuse-base:1 as builder

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
    sqlite-devel \
    fdupes \
    netcfg \
    timezone \
    xz \
    gdbm-devel \
    "

RUN zypper -n refresh && \
    zypper -n in $DEVEL_DEPS

# Build Python from source
ENV PYTHON_VERSION=3.7.6 \
    PYTHON_SHA512=5be022c384728b0c6709d165486f66226d126c521b466ed24d0c4c3c5910ec786a734f4c2c916ac409213c54a492b9fb230fb721c41e3de281676a7e9d8f3171

RUN wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" && \
    echo "$PYTHON_SHA512 python.tar.xz" | sha512sum --strict --check - && \
    mkdir -p /usr/src/python && \
    tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz && \
    rm python.tar.xz

RUN cd /usr/src/python && \
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
    ldconfig

# Remove test files
RUN find /usr/local -depth \
        \( \
            \( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
            -o \
            \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
        \) -exec rm -rf '{}' + && \
    python3 --version

# Some useful symlinks
RUN cd /usr/local/bin && \
    ln -s idle3 idle && \
    ln -s pydoc3 pydoc && \
    ln -s python3 python && \
    ln -s python3-config python-config

# Install pip
ENV PYTHON_PIP_VERSION=19.3.1 \
    PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/ffe826207a010164265d9cc807978e3604d18ca0/get-pip.py \
    PYTHON_GET_PIP_SHA256=b86f36cc4345ae87bfd4f10ef6b2dbfa7a872fbff70608a1e43944d283fd0eee

RUN wget -O get-pip.py "$PYTHON_GET_PIP_URL" && \
    echo "$PYTHON_GET_PIP_SHA256 get-pip.py" | sha256sum --check --strict -

RUN python get-pip.py \
        --disable-pip-version-check \
        --no-cache-dir \
        "pip==$PYTHON_PIP_VERSION" && \
    pip --version


FROM cafapi/opensuse-base:1
RUN zypper -n refresh && \
    zypper -n in libgdbm4

COPY --from=builder /usr/local /usr/local
RUN ldconfig

CMD ["python3"]

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
