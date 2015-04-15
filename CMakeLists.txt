cmake_minimum_required(VERSION 2.8)

project(libseord)

# Detect compiler
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
  set(C_IS_GNU   1)
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  set(C_IS_GNU   1)
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel")
  set(C_IS_INTEL 1)
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  set(C_IS_MSVC  1)
endif()

# Enable C99 mode
if(C_IS_GNU)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
endif()

if(C_IS_INTEL)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -restrict")
endif()

# Enable C++11 mode
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
        message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

# Configuration

set(SERD_VERSION 0.21.1)
set(SORD_VERSION 0.13.0)

include (CheckIncludeFiles)

include(CheckFunctionExists)

check_function_exists(posix_memalign HAVE_POSIX_MEMALIGN)
check_function_exists(posix_fadvise HAVE_POSIX_FADVISE)
check_function_exists(fileno HAVE_FILENO)

configure_file(src/3rd/serd_config.h.in ${CMAKE_CURRENT_BINARY_DIR}/serd_config.h)
configure_file(src/3rd/sord_config.h.in ${CMAKE_CURRENT_BINARY_DIR}/sord_config.h)

# Build

include_directories(
  src/3rd/serd
  src/3rd/sord
  src/3rd/sord/src
  ${CMAKE_CURRENT_BINARY_DIR})

set(LIBSOURCE
  src/3rd/serd/src/uri.c
  src/3rd/serd/src/node.c
  src/3rd/serd/src/env.c
  src/3rd/serd/src/reader.c
  src/3rd/serd/src/serdi.c
  src/3rd/serd/src/writer.c
  src/3rd/serd/src/string.c

  src/3rd/sord/src/zix/hash.c
  src/3rd/sord/src/zix/btree.c
  src/3rd/sord/src/zix/digest.c
  src/3rd/sord/src/syntax.c
  src/3rd/sord/src/sord.c
  )

set(LIBHEADERS
  src/3rd/serd/serd/serd.h
  src/3rd/sord/sord/sord.h
)

add_library(seord SHARED ${LIBSOURCE} ${LIBHEADERS})

add_executable(sordi src/3rd/sord/src/sordi.c ${LIBHEADERS})
target_link_libraries(sordi seord m)

add_executable(serd_test src/3rd/serd/tests/serd_test.c ${LIBHEADERS})
target_link_libraries(serd_test seord m)

add_executable(sord_validate src/3rd/sord/src/sord_validate.c ${LIBHEADERS})
target_link_libraries(sord_validate seord m)

add_executable(sord_test src/3rd/sord/src/sord_test.c ${LIBHEADERS})
target_link_libraries(sord_test seord m)

add_executable(sordmm_test src/3rd/sord/src/sordmm_test.cpp ${LIBHEADERS})
target_link_libraries(sordmm_test seord m)

install(TARGETS seord DESTINATION lib)
install(FILES ${LIBHEADERS} DESTINATION include)
