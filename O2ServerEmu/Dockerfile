# O2Jam Server — built inside Debian 12 (Bookworm) for NAS compatibility
# Reason: Arch Linux's glibc static objects embed AVX opcodes that the NAS
# Celeron N4505 (Goldmont Plus, SSE4.2-only) cannot execute → SIGILL.
# Debian 12's glibc (2.36) and libstdc++ (GCC 12, GLIBCXX_3.4.30) are the
# NAS host's native libraries, so a binary built inside a bookworm container
# runs natively on the NAS with no ISA or version mismatches.
#
# Multi-stage: builder (with g++/boost-dev) → slim runtime (binary + runtime .so).

# ---- Stage 1: build ----
FROM debian:bookworm AS builder

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        g++ make libboost-system-dev libboost-thread-dev && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /build

# O2Jam server source — Boost ASIO TCP server, port 15010
COPY session.cpp utils.cpp utils.h pch.h ./

# Portable ISA flags for the NAS Celeron N4505 (Goldmont Plus):
#   SSE4.2 + SSE2 only, NO AVX / AVX2 / AVX512.
# Debian 12's toolchain (g++ from GCC 12) produces code compatible with
# the NAS's own libc/libstdc++ — no version skew, no AVX in libc objects.
RUN g++ -O2 \
    -march=core2 \
    -msse2 -msse3 -mssse3 -msse4.1 -msse4.2 \
    -mno-avx -mno-avx2 \
    -o O2ServerEmu *.cpp \
    -lboost_system -lboost_thread -lpthread

# ---- Stage 2: lean runtime ----
FROM debian:bookworm-slim

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        libboost-system-dev libboost-thread-dev && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY --from=builder /build/O2ServerEmu /app/O2ServerEmu

# Server reads game assets (Spt/*.spt) relative to CWD (/app).
# Mount a host directory of O2Jam Spt files here:
#   -v <host-path>/Spt:/app/Spt
#
# Logs go to /app/logs (bind-mounted from host for persistence).

CMD ["/app/O2ServerEmu"]
