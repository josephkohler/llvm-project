##===----------------------------------------------------------------------===##
#
# Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
# See https://llvm.org/LICENSE.txt for license information.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
#
##===----------------------------------------------------------------------===##
#
# Build offloading library libomptarget.so.
#
##===----------------------------------------------------------------------===##

libomptarget_say("Building offloading runtime library libomptarget.")

add_llvm_library(omptarget
  SHARED

  api.cpp
  device.cpp
  interface.cpp
  omptarget.cpp
  rtl.cpp
  LegacyAPI.cpp
  PluginManager.cpp
  DeviceImage.cpp

  OpenMP/Mapping.cpp
  OpenMP/InteropAPI.cpp
  OpenMP/OMPT/Callback.cpp


  ADDITIONAL_HEADER_DIRS
  ${LIBOMPTARGET_INCLUDE_DIR}

  LINK_COMPONENTS
  Support
  Object

  LINK_LIBS
  PUBLIC
  omp

  NO_INSTALL_RPATH
)
target_include_directories(omptarget PRIVATE ${LIBOMPTARGET_INCLUDE_DIR})

if (LIBOMP_HAVE_VERSION_SCRIPT_FLAG)
  target_link_libraries(omptarget PRIVATE
    "-Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/exports")
endif()

# Define the TARGET_NAME and DEBUG_PREFIX.
target_compile_definitions(omptarget PRIVATE
  TARGET_NAME=omptarget
  DEBUG_PREFIX="omptarget"
)

# libomptarget.so needs to be aware of where the plugins live as they
# are now separated in the build directory.
set_target_properties(omptarget PROPERTIES
                      POSITION_INDEPENDENT_CODE ON
                      INSTALL_RPATH "$ORIGIN"
                      BUILD_RPATH "$ORIGIN:${CMAKE_CURRENT_BINARY_DIR}/..")
install(TARGETS omptarget LIBRARY COMPONENT omptarget DESTINATION "${OPENMP_INSTALL_LIBDIR}")
