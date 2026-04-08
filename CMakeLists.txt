cmake_minimum_required(VERSION 3.16)

# MpvSDK - prebuilt libmpv for video playback (Windows only)
#
# The prebuilt libmpv-2.dll is a MinGW-compiled binary. We consume it from MSVC
# through its pure C API (extern "C", opaque handles, no CRT sharing), so:
#   - It links correctly with any MSVC toolchain.
#   - It works in all build configurations (Debug/Release/RelWithDebInfo) without
#     debug-vs-release CRT conflicts.
# See README.md for details.
project(acempv
        VERSION 0.0.41
        DESCRIPTION "Precompiled mpv libraries for cross-platform development"
        LANGUAGES C CXX
)

if(NOT WIN32)
    message(STATUS "MpvSDK: Skipping (Windows only)")
    return()
endif()

if(POLICY CMP0111)
    cmake_policy(SET CMP0111 NEW)
endif()

set(MPV_SDK_BASE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/mpv")
set(MPV_SDK_LIB_PATH "${MPV_SDK_BASE_PATH}/lib/win")
set(MPV_SDK_INCLUDE_PATH "${MPV_SDK_BASE_PATH}/include")

find_library(MPV_IMPORT_LIB NAMES mpv
    PATHS "${MPV_SDK_LIB_PATH}" NO_DEFAULT_PATH)

if(NOT MPV_IMPORT_LIB)
    message(FATAL_ERROR "MpvSDK: mpv.lib not found in ${MPV_SDK_LIB_PATH}")
endif()

set(_mpv_dll "${MPV_SDK_LIB_PATH}/libmpv-2.dll")
if(NOT EXISTS "${_mpv_dll}")
    message(FATAL_ERROR "MpvSDK: libmpv-2.dll not found at ${_mpv_dll}")
endif()

add_library(MpvSDK SHARED IMPORTED GLOBAL)
set_target_properties(MpvSDK PROPERTIES
    INTERFACE_INCLUDE_DIRECTORIES "${MPV_SDK_INCLUDE_PATH}"
    IMPORTED_IMPLIB "${MPV_IMPORT_LIB}"
    IMPORTED_LOCATION "${_mpv_dll}"
)

message(STATUS "MpvSDK import lib: ${MPV_IMPORT_LIB}")
message(STATUS "MpvSDK runtime DLL: ${_mpv_dll}")

# 导出配置
include(CMakePackageConfigHelpers)

# 创建版本文件
write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/acempvConfigVersion.cmake"
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion
)

# 创建配置文件
configure_package_config_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake/acempvConfig.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/acempvConfig.cmake"
        INSTALL_DESTINATION "lib/cmake/acempv"
)