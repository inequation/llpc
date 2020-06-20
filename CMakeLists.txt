##
 #######################################################################################################################
 #
 #  Copyright (c) 2017-2020 Advanced Micro Devices, Inc. All Rights Reserved.
 #
 #  Permission is hereby granted, free of charge, to any person obtaining a copy
 #  of this software and associated documentation files (the "Software"), to deal
 #  in the Software without restriction, including without limitation the rights
 #  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 #  copies of the Software, and to permit persons to whom the Software is
 #  furnished to do so, subject to the following conditions:
 #
 #  The above copyright notice and this permission notice shall be included in all
 #  copies or substantial portions of the Software.
 #
 #  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 #  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 #  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 #  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 #  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 #  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
 #  SOFTWARE.
 #
 #######################################################################################################################

project(VKGC LANGUAGES C CXX)

option(AMDLLPC_STANDALONE "Make a stand-alone build of amdllpc, without the whole Vulkan driver" OFF)
if(AMDLLPC_STANDALONE)
    set(XGL_PAL_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../pal")
    set(XGL_ICD_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../xgl/icd")
    set(XGL_METROHASH_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../third_party/metrohash")
    set(XGL_SPVGEN_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../spvgen")
    set(XGL_VKGC_PATH "${CMAKE_CURRENT_SOURCE_DIR}")
    set(CMAKE_MODULE_PATH "${XGL_PAL_PATH}/cmake/Modules")
    set(ICD_BUILD_LLPC ON)

    # This will become the value of LLPC_CLIENT_INTERFACE_MAJOR_VERSION.  It describes the version of the LLPC interface
    # that the ICD supports.  LLPC uses this value to enable backwards-compatibility for older interface versions.  It must
    # be updated on each LLPC promotion after handling all of the interface changes described in llpc.h
    file(STRINGS "${XGL_ICD_PATH}/make/importdefs" LLPC_MAJOR_VERSION REGEX "^ICD_LLPC_CLIENT_MAJOR_VERSION = [0-9]+")

    if(LLPC_MAJOR_VERSION STREQUAL "")
        message(STATUS "Failed to find ICD_LLPC_CLIENT_MAJOR_VERSION")
    else()
        string(REGEX REPLACE "ICD_LLPC_CLIENT_MAJOR_VERSION = " "" LLPC_MAJOR_VERSION ${LLPC_MAJOR_VERSION})
        message(STATUS "Detected ICD_LLPC_CLIENT_MAJOR_VERSION is " ${LLPC_MAJOR_VERSION})
    endif()

    set(LLPC_CLIENT_INTERFACE_MAJOR_VERSION ${LLPC_MAJOR_VERSION})

    # This will become the value of PAL_CLIENT_INTERFACE_MAJOR_VERSION.  It describes the version of the PAL interface
    # that the ICD supports.  PAL uses this value to enable backwards-compatibility for older interface versions.  It must
    # be updated on each PAL promotion after handling all of the interface changes described in palLib.h.
    file(STRINGS "${XGL_ICD_PATH}/make/importdefs" PAL_MAJOR_VERSION REGEX "^ICD_PAL_CLIENT_MAJOR_VERSION = [0-9]+")

    if(PAL_MAJOR_VERSION STREQUAL "")
        message(STATUS "Failed to find ICD_PAL_CLIENT_MAJOR_VERSION")
    else()
        string(REGEX REPLACE "ICD_PAL_CLIENT_MAJOR_VERSION = " "" PAL_MAJOR_VERSION ${PAL_MAJOR_VERSION})
        message(STATUS "Detected ICD_PAL_CLIENT_MAJOR_VERSION is " ${PAL_MAJOR_VERSION})
    endif()

    set(PAL_CLIENT_INTERFACE_MAJOR_VERSION ${PAL_MAJOR_VERSION})
endif()

add_library(vkgc INTERFACE)

### Expose header files ################################################################################################
target_include_directories(vkgc
    INTERFACE
        ${PROJECT_SOURCE_DIR}/include
)

### VKGC build LLPC ################################################################
if(ICD_BUILD_LLPC)
    target_include_directories(vkgc
        INTERFACE
            ${PROJECT_SOURCE_DIR}/llpc/include
            ${PROJECT_SOURCE_DIR}/lgc/include
    )

    add_subdirectory(llpc ${PROJECT_BINARY_DIR}/llpc)

    target_link_libraries(vkgc INTERFACE llpc)
endif()

