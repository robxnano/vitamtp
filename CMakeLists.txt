# Copyright (C) 2025 Rob Hall
# SPDX-License-Identifier: BSD-3-Clause

cmake_minimum_required(VERSION 3.10...4.0)
project(VitaMTP LANGUAGES C VERSION 2.5.9)

include(CheckLinkerFlag)
include(CheckSymbolExists)
include(FindPkgConfig)
include(FindThreads)
include(FindIconv)
include(FindIntl)
include(GNUInstallDirs)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_COLOR_DIAGNOSTICS ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

option(ENABLE_DOXYGEN "Enable Doxygen documentation" OFF)
option(ENABLE_ICONV "Enable iconv support" ON)
option(ENABLE_USB "Enable libusb support" ON)
option(ENABLE_WIRELESS "Enable wireless support" ON)
option(INSTALL_UDEV_RULES "Install udev rules file" ON)
option(BUILD_SHARED_LIBS "Build shared library" ON)

add_library(vitamtp)
add_library(VitaMTP::VitaMTP ALIAS vitamtp)
target_compile_options(vitamtp PRIVATE -Wall -Wextra -Wno-unused-parameter -Wno-parentheses)
set_target_properties(vitamtp PROPERTIES VERSION 5.0.1 SOVERSION 5)

set(PACKAGE ${PROJECT_NAME})
set(PACKAGE_NAME ${PROJECT_NAME})

check_include_file(limits.h HAVE_LIMITS_H)
target_include_directories(vitamtp PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/src)
target_include_directories(vitamtp PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
                                          $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
set_target_properties(vitamtp PROPERTIES PUBLIC_HEADER src/vitamtp.h)

if(WIN32)
    target_link_libraries(vitamtp PRIVATE ws2_32)
    list(APPEND LIBS_PRIVATE -lws2_32)
    target_compile_options(vitamtp PRIVATE -mno-ms-bitfields)
endif()

if(NOT MSVC)
    set(LINK_VERSION_SCRIPT "-Wl,--version-script,${CMAKE_CURRENT_SOURCE_DIR}/src/libvitamtp.map")
    check_linker_flag(C ${LINK_VERSION_SCRIPT} LINKER_SUPPORTS_VERSION_SCRIPT)
    if(LINKER_SUPPORTS_VERSION_SCRIPT)
        set_property(TARGET vitamtp PROPERTY OBJECT_DEPENDS src/libvitamtp.map)
        target_link_options(vitamtp PRIVATE ${LINK_VERSION_SCRIPT})
    endif()
endif()

pkg_check_modules(libxml-2.0 REQUIRED IMPORTED_TARGET libxml-2.0)
target_link_libraries(vitamtp PRIVATE PkgConfig::libxml-2.0 Threads::Threads)
list(APPEND REQUIRES_PRIVATE libxml-2.0)
set(HAVE_LIBXML2 1)

if(ENABLE_ICONV AND Iconv_FOUND)
    set(HAVE_ICONV 1)
    target_link_libraries(vitamtp PRIVATE Iconv::Iconv)
    list(APPEND LIBS_PRIVATE ${Iconv_LIBRARIES})
endif()

if(Intl_FOUND)
    set(ENABLE_NLS 1)
    target_link_libraries(vitamtp PRIVATE Intl::Intl)
endif()

if(ENABLE_USB)
    pkg_check_modules(libusb-1.0 REQUIRED IMPORTED_TARGET libusb-1.0)
    target_compile_definitions(vitamtp PRIVATE PTP_USB_SUPPORT)
    target_link_libraries(vitamtp PRIVATE PkgConfig::libusb-1.0)
    list(APPEND REQUIRES_PRIVATE libusb-1.0)
endif()

if(ENABLE_WIRELESS)
    if(WIN32)
        check_include_file(winsock2.h HAVE_WINSOCK2_H)
        if(NOT HAVE_WINSOCK2_H)
            message(FATAL_ERROR "The winsock2.h header was not found")
        endif()
    endif()
    target_compile_definitions(vitamtp PRIVATE PTP_IP_SUPPORT)
endif()

configure_file(config.h.in ${CMAKE_CURRENT_BINARY_DIR}/src/config.h)

if(PROJECT_IS_TOP_LEVEL)
    list(JOIN REQUIRES_PRIVATE " " PC_REQUIRES_PRIVATE)
    list(JOIN LIBS_PRIVATE " " PC_LIBS_PRIVATE)
    list(JOIN REQUIRES " " PC_REQUIRES)
    list(JOIN VITAMTP_CFLAGS " " PC_CFLAGS)
    configure_file(libvitamtp.pc.in ${CMAKE_CURRENT_BINARY_DIR}/libvitamtp.pc @ONLY)
    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libvitamtp.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
endif()

if(INSTALL_UDEV_RULES)
    install(FILES packaging/udev/60-psvita.rules DESTINATION lib/udev/rules.d)
endif()

add_subdirectory(src)

if(ENABLE_DOXYGEN)
    add_subdirectory(doc)
endif()

if(PROJECT_IS_TOP_LEVEL OR BUILD_SHARED_LIBS)
    install(TARGETS vitamtp
            EXPORT VitaMTPTargets
            RUNTIME LIBRARY PUBLIC_HEADER)
endif()
