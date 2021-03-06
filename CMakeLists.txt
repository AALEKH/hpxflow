#////////////////////////////////////////////////////////////////////////////////
#//  Copyright (c) 2016 Aalekh Nigam
#//  Copyright (c) 2016 Patrick Diehl
#//  Distributed under the Boost Software License, Version 1.0. (See accompanying
#//  file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
#////////////////////////////////////////////////////////////////////////////////


cmake_minimum_required(VERSION 3.0.0 FATAL_ERROR)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING
        "Configuration type (one of Debug, RelWithDebInfo, Release, MinSizeRel)")
endif()

# This project is C++ based.
project(hpxflow CXX)

# The version number of the project
set(VERSION_MAJOR 1 CACHE STRING "Project major version number")
set(VERSION_MINOR 0 CACHE STRING "Project minor version number")

if(HPX_ROOT)
    set(HPX_DIR "${HPX_ROOT}/lib/cmake/HPX")
endif()
set (HPX_IGNORE_COMPILER_COMPATIBILITY  ON)
#add -std=gnu++11 to the debug flags
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    if (CMAKE_COMPILER_IS_GNUCC)
         SET(CMAKE_CXX_FLAGS_DEBUG "-ggdb -D_DEBUG -O0 -std=gnu++11" CACHE STRING)
    endif()
endif()

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/")

########################################################################
# Include hpx
# Instruct cmake to find the HPX settings

find_package(MYSQL)

# Instructions to include mysql
if(MYSQL_ROOT)
    include_directories(${MYSQL_ROOT})
endif()

################################################################################

find_package(HPX REQUIRED)

if(NOT (HPX_BUILD_TYPE STREQUAL CMAKE_BUILD_TYPE))
    message("HPX_BUILD_TYPE: " ${HPX_BUILD_TYPE} "CMAKE_BUILD_TYPE: " ${CMAKE_BUILD_TYPE})
    message(FATAL_ERROR "Build types used for HPX and this library have to match")
endif()


# inherit varios setting from HPXConfig
set(CMAKE_DEBUG_POSTFIX ${HPX_DEBUG_POSTFIX})

# Adjust auxiliary build settings for HPX to work properly
if(NOT MSVC)
    set(HPX_RPATH
        "${PROJECT_BINARY_DIR}/lib/hpx:${HPX_RPATH}:${CMAKE_INSTALL_PREFIX}/lib/hpx")
else()
    # enable solution folders for MSVC
    set_property(GLOBAL PROPERTY USE_FOLDERS ON)
    if(HPX_NO_INSTALL)
      set(CMAKE_INSTALL_PREFIX "${PROJECT_BINARY_DIR}/$(Configuration)" CACHE PATH
          "Prefix prepended to install directories." FORCE)
    endif()
endif()

file( GLOB_RECURSE sources ./src/algorithm/*.cpp  )
file( GLOB_RECURSE sources_t  ./src/core_layer/*.cpp  )
file( GLOB_RECURSE sources_f  ./src/dataflow/*.cpp  )
file( GLOB_RECURSE sources_s  ./src/pipeline/*.cpp  )

file( GLOB_RECURSE headers ./src/algorithm/*.hpp  )
file( GLOB_RECURSE headers_t  ./src/core_layer/*.hpp )
file( GLOB_RECURSE headers_f  ./src/dataflow/*.hpp  )
file( GLOB_RECURSE headers_s  ./src/pipeline/*.hpp  )

add_hpx_component(hpxflow
        SOURCES ${sources} ${sources_t} ${sources_f} ${sources_s}
		COMPONENT_DEPENDENCIES iostreams
		DEPENDENCIES ${MYSQL_LIBRARIES}
		)

find_package(Boost 1.58.0 COMPONENTS date_time filesystem program_options regex serialization system thread REQUIRED)

########################################################################
# configurations
hpx_option(HPXFLOW_BUILD_EXAMPLES BOOL "Build HPX examples (default: OFF)" ON )
hpx_option(HPXFLOW_BUILD_TESTS BOOL "Build HPX tests (default: OFF)" OFF ADVANCED)
hpx_option(HPXFLOW_WITH_DOCUMENTATION BOOL "The HPX documentation toolchain is available (default OFF)." OFF)

#endif()

if(MSVC)
  hpx_add_compile_flag(-Ox CONFIGURATIONS Release LANGUAGES CXX C)
  hpx_add_compile_flag(-RTC1 CONFIGURATIONS Debug LANGUAGES CXX C)

  # VS2012 and above has a special flag for improving the debug experience by
  # adding more symbol information to the build (-d2Zi)
  hpx_add_compile_flag(-d2Zi+ CONFIGURATIONS RelWithDebInfo)

  # VS2013 and above know how to do link time constant data segment folding
  # VS2013 update 2 and above know how to remove debug information for
  #     non-referenced functions and data (-Zc:inline)
  if(MSVC12 OR MSVC13)
    hpx_add_compile_flag(-Zc:inline)
    hpx_add_compile_flag(-Gw CONFIGURATIONS Release RelWithDebInfo MinSizeRelease)
    hpx_add_compile_flag(-Zo CONFIGURATIONS RelWithDebInfo)
  endif()
  hpx_add_compile_flag(-EHsc)       # Exceptions
  hpx_add_compile_flag(-GR)         # Runtime type information
  hpx_add_compile_flag(-MP)         # Multiprocessor build
  hpx_add_compile_flag(-bigobj)     # Increase the maximum size of object file sections
endif()

include_directories(${PROJECT_SOURCE_DIR})

if(HPXFLOW_BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

if(HPXFLOW_WITH_DOCUMENTATION)
    find_package(Doxygen)
    add_subdirectory(doc)
endif()

#if(HPXFLOW_WITH_BENCHMARK)
#   add_hpx_pseudo_target(benchmark)
#   include_directories(benchmark)
#    add_subdirectory(benchmark)
#endif()

