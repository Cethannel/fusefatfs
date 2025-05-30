cmake_minimum_required(VERSION 3.12)
project(fusefatfs
    VERSION 0.2
    DESCRIPTION "fuse/vuos module for fat (12/16/32/exfat)"
    HOMEPAGE_URL "https://github.com/virtualsquare/fusefatfs"
    LANGUAGES C)

include(GNUInstallDirs)
include(CheckIncludeFile)
include_directories(${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

find_package(PkgConfig)
pkg_check_modules(FUSE fuse3)
if(NOT FUSE_FOUND)
	pkg_check_modules(FUSE REQUIRED fuse)
endif()
string(REGEX REPLACE "\\..*" "" FUSE_VERSION ${FUSE_VERSION})

set(CMAKE_REQUIRED_DEFINITIONS -D_FILE_OFFSET_BITS=64)

configure_file(config.h.in config.h @ONLY)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_FORTIFY_SOURCE=2 -O2 -pedantic -Wall -Wextra")

add_definitions(-D_GNU_SOURCE -D_FILE_OFFSET_BITS=64 ${FUSE_CFLAGS} -DFUSE=${FUSE_VERSION})

add_executable(fusefatfs fusefatfs.c fftable.c diskio.c ff.c ffunicode.c)
target_link_libraries(fusefatfs ${FUSE_LIBRARIES})
install(TARGETS fusefatfs
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

add_library(vufusefatfs SHARED fusefatfs.c fftable.c diskio.c ff.c ffunicode.c)
set_target_properties(vufusefatfs PROPERTIES PREFIX "")
install(TARGETS vufusefatfs
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/vu/modules)

add_subdirectory(man)

add_custom_target(uninstall
  "${CMAKE_COMMAND}" -P "${PROJECT_SOURCE_DIR}/Uninstall.cmake"
  )
