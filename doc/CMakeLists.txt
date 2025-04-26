# Copyright (C) 2025 Rob Hall
# SPDX-License-Identifier: BSD-3-Clause

find_package(Doxygen)
if(DOXYGEN_FOUND)
    set(QUIET YES)
    doxygen_add_docs(VitaMTP
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${PROJECT_SOURCE_DIR}/src
    )
endif()