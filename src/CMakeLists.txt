# Copyright (C) 2025 Rob Hall
# SPDX-License-Identifier: BSD-3-Clause

target_sources(vitamtp PRIVATE
    datautils.c
    device.c
    ptp.c
    ptp.h
    ptp_defines.h
    usb.c
    vitamtp.c
    vitamtp.h
    wireless.c
)

check_symbol_exists(bswap_32 byteswap.h HAS_BSWAP_32)
check_symbol_exists(le32toh machine/endian.h HAS_LE32TOH)
check_symbol_exists(ntohl arpa/inet.h HAS_NTOHL_ARPA_INET)
check_symbol_exists(ntohl netinet/in.h HAS_NTOHL_NETINET_IN)
check_symbol_exists(swap32 machine/endian.h HAS_SWAP32)
check_symbol_exists(__byte_swap_long sys/types.h HAS_BYTE_SWAP_LONG)
check_symbol_exists(NXSwapLong machine/byte_order.h HAS_NXSWAPLONG)
if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86|i.86|x86_64|AMD64|amd64)")
    set(CAN_CAST_BYTES_TO_MEM 1)
endif()

configure_file(gphoto2-endian.h.in ${CMAKE_CURRENT_BINARY_DIR}/gphoto2-endian.h)

if(WIN32)
    target_sources(vitamtp PRIVATE asprintf.c socketpair.c)
endif()
