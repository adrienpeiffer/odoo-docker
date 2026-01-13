# syntax=docker/dockerfile:1.4

#######################################################################################
# base stage, with the non-python runtime dependencies
#
FROM ghcr.io/acsone/odoo-bedrock:13.0-py310-jammy-latest as base

# Install apt runtime dependencies.
# - postgresql-client for comfort in the shell container and for db dump to work
# - expect to have unbuffer in CI
RUN set -e \
  && apt update \
  && apt -y install --no-install-recommends \
       postgresql-client \
       expect \
       git \
       openssh-client \
       libcairo2 \
       sudo \
  && apt -y clean \
  && rm -rf /var/lib/apt/lists/*

#######################################################################################
# builds-deps stage, where we download requirements-build.txt,
# and install tools necessary to build source distributions.
#

FROM base as build-deps

RUN set -e \
  && apt update \
  && apt -y install --no-install-recommends \
       python3.10-dev \
       build-essential \
       libpq-dev \
       pkg-config \
       ca-certificates \
       python3-dev \
      ldap-utils \
      libldap2-dev \
      libsasl2-dev \
      libcairo2-dev \
  && apt -y clean \
  && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

RUN pip install setuptools==57.5.0 wheel
RUN pip wheel --no-build-isolation --no-cache-dir --wheel-dir=/build vatnumber==1.2
RUN pip wheel --no-deps --wheel-dir=/build -r requirements.txt

FROM base as dependencies

# Install python dependencies we built in the build stage.
# Use --no-deps and --no-index to be sure to not download anything else.

RUN --mount=type=bind,target=/build,source=/build,from=build-deps \
  pip install --no-deps --no-index /build/*.whl