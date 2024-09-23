#!/usr/bin/cmake
cmake_minimum_required (VERSION 3.18)

# -------------------------------------------------------------------------------------------------

project(
  OpusEnc
  VERSION 0.12
  DESCRIPTION "Add-on for libopus to encode audio data into the Opus codec and file format"
)

# This sets a bunch of compile flags and defined ${NUCLEX_COMPILER_TAG} to
# say something like linux-gcc-13.2-amd64-debug. You should have this directory
# if you do a full clone of a project that is using this third-party library build.
include("../../build-system/cmake/cplusplus.cmake")

# Add OPUS as a sub-project, this is a library built on top of libopus
if(NOT (TARGET Opus))
  add_subdirectory(
    ${PROJECT_SOURCE_DIR}/../nuclex-opus
    ${CMAKE_BINARY_DIR}/nuclex-opus
  )
endif()

# -------------------------------------------------------------------------------------------------

# TODO: Effin' CMake again. Can't this abomination of a useless build system just die?
#  If downloads the file. It complains about the checksum if it's wrong.
# Then it writes a BAD status into DOWNLOAD_STATUS. And leaves ERROR_MESSAGE empty.
# Completely useless. Weird, custom build language with side effects as the primary
# means of returning results and that probably changes between versions. Not fixing this.
if(NOT EXISTS ${PROJECT_SOURCE_DIR}/build)
  message(STATUS "Build directory not present, downloading sources")

  file(
    DOWNLOAD https://github.com/xiph/opusenc/releases/download/v0.12/opusenc-0.12.tar.gz
    ${PROJECT_SOURCE_DIR}/downloads/opusenc-0.12.tar.gz
    SHOW_PROGRESS
    EXPECTED_HASH SHA256=7875d49596286055b52439ed42f044bd8ad426aa4cc5aabd96bfe7abb971d5e8
  )
  list(GET DOWNLOAD_STATUS 0 STATUS_CODE)
  if(NOT ${STATUS_CODE} EQUAL 0)
    list(GET DOWNLOAD_STATUS 1 ERROR_MESSAGE)
    file(REMOVE ${PROJECT_SOURCE_DIR}/downloads/opusenc-0.12.tar.gz)
    message(FATAL_ERROR "Could not download OpusEnc: ${ERROR_MESSAGE}")
  endif()

  message(STATUS "Extracting OpusEnc sources")
  file(
    ARCHIVE_EXTRACT
    INPUT ${PROJECT_SOURCE_DIR}/downloads/opusenc-0.12.tar.gz
    DESTINATION ${CMAKE_BINARY_DIR}/extract
  )
  file(
    RENAME
    ${CMAKE_BINARY_DIR}/extract/opusenc-0.12
    ${PROJECT_SOURCE_DIR}/build
  )
endif()

# -------------------------------------------------------------------------------------------------

configure_file(
  "${PROJECT_SOURCE_DIR}/config.h"
  "${PROJECT_SOURCE_DIR}/build/src/config.h"
  COPYONLY
)

# -------------------------------------------------------------------------------------------------

set(
  sourceFiles
  "build/src/resample.c"
  "build/src/opus_header.c"
  "build/src/picture.c"
  "build/src/unicode_support.c"
  "build/src/opusenc.c"
  "build/src/ogg_packer.c"
)
set(
  headerFiles
  "build/include/opusenc.h"
)

# -------------------------------------------------------------------------------------------------

add_library(OpusEnc STATIC)
add_library(OpusEnc::Static ALIAS OpusEnc)

target_compile_definitions(
  OpusEnc
  PUBLIC OUTSIDE_SPEEX
  PUBLIC RANDOM_PREFIX=private_
  PRIVATE OPUSENC_BUILD
  PRIVATE HAVE_CONFIG_H
)

target_include_directories(
  OpusEnc
  PUBLIC "build/include"
)

target_link_libraries(
  OpusEnc
  PRIVATE Opus
)

target_sources(
  OpusEnc
  PUBLIC ${headerFiles}
  PRIVATE ${sourceFiles}
)

set_target_properties(OpusEnc PROPERTIES PREFIX "")
set_target_properties(OpusEnc PROPERTIES OUTPUT_NAME "opusenc")

# -------------------------------------------------------------------------------------------------

install(
  TARGETS OpusEnc
  ARCHIVE DESTINATION ${PROJECT_SOURCE_DIR}/bin/${NUCLEX_COMPILER_TAG}
)
