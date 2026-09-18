FROM ruby:4.0.2-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    git \
    pkg-config \
    libyaml-dev \
    libsqlite3-dev \
    zlib1g-dev \
    curl \
    unzip \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Needed to run tests that exercise the google-protobuf gem, mirrors .github/workflows/ci.yml
RUN curl -OL https://github.com/google/protobuf/releases/download/v25.3/protoc-25.3-linux-x86_64.zip \
    && unzip protoc-25.3-linux-x86_64.zip -d /usr/local \
    && rm protoc-25.3-linux-x86_64.zip

WORKDIR /workspace
