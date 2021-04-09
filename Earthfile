# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

FROM alpine:3.13
WORKDIR /arrow

# Declaring these makes Earthly pick them up from .env.
ARG ARCH
ARG ARCH_ALIAS
ARG ARCH_SHORT_ALIAS
ARG ULIMIT_CORE
ARG REPO
ARG CUDA
ARG DEBIAN
ARG UBUNTU
ARG FEDORA
ARG PYTHON
ARG LLVM
ARG CLANG_TOOLS
ARG RUST
ARG GO
ARG NODE
ARG MAVEN
ARG JDK
ARG NUMPY
ARG PANDAS
ARG DASK
ARG TURBODBC
ARG KARTOTHEK
ARG HDFS
ARG SPARK
ARG DOTNET
ARG R
ARG ARROW_R_DEV
ARG R_ORG
ARG R_IMAGE
ARG R_TAG
ARG DEVTOOLSET_VERSION

# Used for the manylinux and windows wheels
ARG VCPKG=fced4bef1606260f110d74de1ae1975c2b9ac549

conda-cpp:
    FROM ./ci+conda-cpp-deps
    # Alternative: use .dockerfile.
    #FROM DOCKERFILE +dockerfile-context/ --dockerfile=conda-cpp
    WORKDIR /arrow
    DO +CCACHE_ENV
    ENV ARROW_BUILD_BENCHMARKS="ON"
    ENV ARROW_ENABLE_TIMING_TESTS=
    ENV ARROW_MIMALLOC="ON"
    ENV ARROW_USE_LD_GOLD="ON"
    ENV ARROW_USE_PRECOMPILED_HEADERS="ON"
    # TODO: Minimize this COPY.
    COPY --dir ./* /arrow/
    RUN mkdir -p /build
    RUN --entrypoint --mount=type=cache,target=/ccache -- \
        "/arrow/ci/scripts/cpp_build.sh /arrow /build true ||
        (
            echo \"=============== CMakeOutput.log ===============\";
            cat /build/cpp/CMakeFiles/CMakeOutput.log;
            echo \"=============== CMakeError.log ===============\";
            cat /build/cpp/CMakeFiles/CMakeError.log;
            exit 1
        )"
    SAVE ARTIFACT /build/cpp/* AS LOCAL ./build/cpp/

conda-cpp-test:
    FROM +conda-cpp
    RUN --entrypoint "/arrow/ci/scripts/cpp_test.sh /arrow /build"

conda-python:
    FROM ./ci+conda-python-deps
    WORKDIR /arrow
    COPY --dir +conda-cpp/* /build/cpp/
    # TODO: Minimize this COPY.
    COPY --dir ./* /arrow/
    RUN --entrypoint --mount=type=cache,target=/ccache -- \
        "PYTHON=python /arrow/ci/scripts/python_build.sh /arrow /build"
    SAVE ARTIFACT /build/python/* AS LOCAL ./build/python/

conda-python-test:
    FROM +conda-python
    RUN --entrypoint "/arrow/ci/scripts/python_test.sh /arrow"

dockerfile-context:
    COPY --dir ./* ./
    ARG dockerfile
    COPY ./ci/docker+dockerfiles/"$dockerfile".dockerfile ./Dockerfile
    SAVE ARTIFACT ./*

CCACHE_ENV:
    COMMAND
    ENV CCACHE_COMPILERCHECK="content"
    ENV CCACHE_COMPRESS="1"
    ENV CCACHE_COMPRESSLEVEL="6"
    ENV CCACHE_MAXSIZE="500M"
    ENV CCACHE_DIR="/ccache"
