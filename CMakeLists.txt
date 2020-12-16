cmake_minimum_required(VERSION 3.5)

project(scs
  LANGUAGES C
  VERSION 2.1.2)

# Defines the CMAKE_INSTALL_LIBDIR, CMAKE_INSTALL_BINDIR and many other useful macros.
# See https://cmake.org/cmake/help/latest/module/GNUInstallDirs.html
include(GNUInstallDirs)

# Control where libraries and executables are placed during the build.
# With the following settings executables are placed in <the top level of the
# build tree>/bin and libraries/archives in <top level of the build tree>/lib.
# This is particularly useful to run ctests on libraries built on Windows
# machines: tests, which are executables, are placed in the same folders of
# dlls, which are treated as executables as well, so that they can properly
# find the libraries to run. This is a because of missing RPATH on Windows.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")

# To build shared libraries in Windows, we set CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS to TRUE.
# See https://cmake.org/cmake/help/v3.4/variable/CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS.html
# See https://blog.kitware.com/create-dlls-on-windows-without-declspec-using-new-cmake-export-all-feature/
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

# Under MSVC, we set CMAKE_DEBUG_POSTFIX to "d" to add a trailing "d" to library
# built in debug mode. In this Windows user can compile, build and install the
# library in both Release and Debug configuration avoiding naming clashes in the
# installation directories.
if(MSVC)
  set(CMAKE_DEBUG_POSTFIX "d")
endif()

# Build position independent code.
# Position Independent Code (PIC) is commonly used for shared libraries so that
# the same shared library code can be loaded in each program address space in a
# location where it will not overlap with any other uses of such memory.
# In particular, this option avoids problems occurring when a process wants to
# load more than one shared library at the same virtual address.
# Since shared libraries cannot predict where other shared libraries could be
# loaded, this is an unavoidable problem with the traditional shared library
# concept.
# Generating position-independent code is often the default behavior for most
# modern compilers.
# Moreover linking a static library that is not built with PIC from a shared
# library will fail on some compiler/architecture combinations.
# Further details on PIC can be found here:
# https://eli.thegreenplace.net/2011/11/03/position-independent-code-pic-in-shared-libraries/
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# Disable C and C++ compiler extensions.
# C/CXX_EXTENSIONS are ON by default to allow the compilers to use extended
# variants of the C/CXX language.
# However, this could expose cross-platform bugs in user code or in the headers
# of third-party dependencies and thus it is strongly suggested to turn
# extensions off.
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_CXX_EXTENSIONS OFF)

# We use
# - InstallBasicPackageFiles (http://robotology.github.io/ycm/gh-pages/v0.8/module/InstallBasicPackageFiles.html)
# - AddUninstallTarget (http://robotology.github.io/ycm/gh-pages/v0.8/module/AddUninstallTarget.html)
# - AddInstallRPATHSupport (http://robotology.github.io/ycm/gh-pages/v0.8/module/AddInstallRPATHSupport.html)
# from YCM. Files are under the cmake subdirectory.
# See https://github.com/robotology/ycm/
# If you don't want to ship these files with your code (recommended), you can
# instead depend on YCM.
# In this case replace the following line with
# find_package(YCM REQUIRED)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

### Options
# Shared/Dynamic or Static library?
option(BUILD_SHARED_LIBS "Build libraries as shared as opposed to static" ON)

# Enable RPATH support for installed binaries and libraries
include(AddInstallRPATHSupport)
add_install_rpath_support(BIN_DIRS "${CMAKE_INSTALL_FULL_BINDIR}"
  LIB_DIRS "${CMAKE_INSTALL_FULL_LIBDIR}"
  INSTALL_NAME_DIR "${CMAKE_INSTALL_FULL_LIBDIR}"
  USE_LINK_PATH)

# Encourage user to specify a build type (e.g. Release, Debug, etc.), otherwise set it to Release.
if(NOT CMAKE_CONFIGURATION_TYPES)
  if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY VALUE "Release")
  endif()
endif()


# Build test related commands?
option(BUILD_TESTING "Create tests using CMake" OFF)
if(BUILD_TESTING)
  enable_testing()
endif()

## Clone scs repository
include(FetchContent)
FetchContent_Declare(
  scs
  GIT_REPOSITORY https://github.com/cvxgrp/scs.git
  GIT_TAG        v${PROJECT_VERSION}
  )

FetchContent_MakeAvailable(scs)


set(LINSYS linsys)
set(DIRSRC ${LINSYS}/cpu/direct)
set(INDIRSRC ${LINSYS}/cpu/direct)
set(EXTERNAL ${LINSYS}/external)

set(${PROJECT_NAME}_SRC
  ${scs_SOURCE_DIR}/src/aa.c
  ${scs_SOURCE_DIR}/src/cones.c
  ${scs_SOURCE_DIR}/src/ctrlc.c
  ${scs_SOURCE_DIR}/src/linalg.c
  ${scs_SOURCE_DIR}/src/normalize.c
  ${scs_SOURCE_DIR}/src/rw.c
  ${scs_SOURCE_DIR}/src/scs.c
  ${scs_SOURCE_DIR}/src/scs_version.c
  ${scs_SOURCE_DIR}/src/util.c
  ${scs_SOURCE_DIR}/${LINSYS}/amatrix.c)

set(${PROJECT_NAME}_HDR
  ${scs_SOURCE_DIR}/include/aa.h
  ${scs_SOURCE_DIR}/include/cones.h
  ${scs_SOURCE_DIR}/include/ctrlc.h
  ${scs_SOURCE_DIR}/include/glbopts.h
  ${scs_SOURCE_DIR}/include/linalg.h
  ${scs_SOURCE_DIR}/include/linsys.h
  ${scs_SOURCE_DIR}/include/normalize.h
  ${scs_SOURCE_DIR}/include/rw.h
  ${scs_SOURCE_DIR}/include/scs.h
  ${scs_SOURCE_DIR}/include/scs_blas.h
  ${scs_SOURCE_DIR}/include/util.h
  ${scs_SOURCE_DIR}/${LINSYS}/amatrix.h)

# get all the c file in amd/external
file(GLOB ${PROJECT_NAME}_AMD_EXTERNAL_SRC
  ${scs_SOURCE_DIR}/${EXTERNAL}/amd/*.c
  )

file(GLOB ${PROJECT_NAME}_AMD_EXTERNAL_HDR
  ${scs_SOURCE_DIR}/${EXTERNAL}/amd/*.h
  )

file(GLOB ${PROJECT_NAME}_LDL_EXTERNAL_HDR
  ${scs_SOURCE_DIR}/${EXTERNAL}/qdldl/*.h
  )


set(${PROJECT_NAME}_DIRECT ${PROJECT_NAME}dir)
add_library(${${PROJECT_NAME}_DIRECT}
  ${${PROJECT_NAME}_HDR}
  ${${PROJECT_NAME}_SRC}
  ${scs_SOURCE_DIR}/${DIRSRC}/private.c
  ${scs_SOURCE_DIR}/${DIRSRC}/private.h
  ${scs_SOURCE_DIR}/${EXTERNAL}/qdldl/qdldl.c
  ${${PROJECT_NAME}_AMD_EXTERNAL_SRC}
  ${${PROJECT_NAME}_AMD_EXTERNAL_HDR}
  ${${PROJECT_NAME}_LDL_EXTERNAL_HDR})

target_include_directories(${${PROJECT_NAME}_DIRECT} PRIVATE
  "$<BUILD_INTERFACE:${scs_SOURCE_DIR}/${EXTERNAL}/amd;${scs_SOURCE_DIR}/${EXTERNAL}/qdldl;${scs_SOURCE_DIR}/${DIRSRC};${scs_SOURCE_DIR}/${LINSYS}>")

target_include_directories(${${PROJECT_NAME}_DIRECT} PUBLIC
  "$<BUILD_INTERFACE:${scs_SOURCE_DIR}/include>"
  "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/scs>")


set_property(GLOBAL APPEND PROPERTY scs_TARGETS ${${PROJECT_NAME}_DIRECT})
target_compile_definitions(${${PROJECT_NAME}_DIRECT} PRIVATE -DUSE_LAPACK)

target_link_libraries(${${PROJECT_NAME}_DIRECT} PRIVATE
  m
  blas
  lapack)


set_target_properties(${${PROJECT_NAME}_DIRECT} PROPERTIES
  VERSION ${scs_VERSION}
  PUBLIC_HEADER "${${PROJECT_NAME}_HDR}")

add_library(scs::${${PROJECT_NAME}_DIRECT} ALIAS ${${PROJECT_NAME}_DIRECT})

install(TARGETS ${${PROJECT_NAME}_DIRECT}
  EXPORT  ${PROJECT_NAME}
  COMPONENT runtime
  LIBRARY          DESTINATION "${CMAKE_INSTALL_LIBDIR}"                            COMPONENT shlib
  ARCHIVE          DESTINATION "${CMAKE_INSTALL_LIBDIR}"                            COMPONENT lib
  RUNTIME          DESTINATION "${CMAKE_INSTALL_BINDIR}"                            COMPONENT bin
  PUBLIC_HEADER    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/scs")


set(${PROJECT_NAME}_INDIRECT ${PROJECT_NAME}indir)
add_library(${${PROJECT_NAME}_INDIRECT}
  ${${PROJECT_NAME}_HDR}
  ${${PROJECT_NAME}_SRC}
  ${scs_SOURCE_DIR}/${INDIRSRC}/private.c
  ${${${PROJECT_NAME}_INDIRECT}_HDR}
  ${scs_SOURCE_DIR}/${EXTERNAL}/qdldl/qdldl.c
  ${${PROJECT_NAME}_AMD_EXTERNAL_SRC}
  ${${PROJECT_NAME}_AMD_EXTERNAL_HDR}
  ${${PROJECT_NAME}_LDL_EXTERNAL_SRC}
  )

set_target_properties(${${PROJECT_NAME}_INDIRECT} PROPERTIES
  VERSION ${scs_VERSION}
  PUBLIC_HEADER "${${${PROJECT_NAME}_INDIRECT}_HDR}")

target_include_directories(${${PROJECT_NAME}_INDIRECT} PRIVATE
  "$<BUILD_INTERFACE:${scs_SOURCE_DIR}/${EXTERNAL}/amd;${CMAKE_CURRENT_SOURCE_INDIR}/scs/${EXTERNAL}/qdldl;${scs_SOURCE_DIR}/${INDIRSRC};${scs_SOURCE_DIR}/${LINSYS}>")

target_include_directories(${${PROJECT_NAME}_INDIRECT} PUBLIC
  "$<BUILD_INTERFACE:${scs_SOURCE_DIR}/include>"
  "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/scs>")

set_property(GLOBAL APPEND PROPERTY scs_TARGETS ${${PROJECT_NAME}_INDIRECT})
target_compile_definitions(${${PROJECT_NAME}_INDIRECT} PRIVATE -DUSE_LAPACK)

target_link_libraries(${${PROJECT_NAME}_INDIRECT} PRIVATE
  m
  blas
  lapack)

install(TARGETS ${${PROJECT_NAME}_INDIRECT}
  EXPORT  ${PROJECT_NAME}
  COMPONENT runtime
  LIBRARY       DESTINATION "${CMAKE_INSTALL_LIBDIR}"                            COMPONENT shlib
  ARCHIVE       DESTINATION "${CMAKE_INSTALL_LIBDIR}"                            COMPONENT lib
  RUNTIME       DESTINATION "${CMAKE_INSTALL_BINDIR}"                            COMPONENT bin
  PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/scs")

add_library(scs::${${PROJECT_NAME}_INDIRECT} ALIAS ${${PROJECT_NAME}_INDIRECT})

include(InstallBasicPackageFiles)
install_basic_package_files(${PROJECT_NAME}
  NAMESPACE scs::
  VERSION ${${PROJECT_NAME}_VERSION}
  TARGETS_PROPERTY scs_TARGETS
  COMPATIBILITY SameMajorVersion
  VARS_PREFIX ${PROJECT_NAME}
  NO_CHECK_REQUIRED_COMPONENTS_MACRO)

## Add the tests
if(BUILD_TESTING)
  add_executable(run_tests_direct  ${scs_SOURCE_DIR}/test/run_tests.c)
  target_link_libraries(run_tests_direct PRIVATE
    scs::scsdir)
  target_include_directories(run_tests_direct PRIVATE
    "$<BUILD_INTERFACE:${scs_SOURCE_DIR}/test;${scs_SOURCE_DIR}/include;${scs_SOURCE_DIR}/${LINSYS}>")

  add_test(NAME run_tests_direct
    COMMAND run_tests_direct
    WORKING_DIRECTORY ${scs_SOURCE_DIR}
    )

  add_executable(run_tests_indirect  ${scs_SOURCE_DIR}/test/run_tests.c)
  target_link_libraries(run_tests_indirect PRIVATE
    scs::scsindir)
  target_include_directories(run_tests_indirect PRIVATE
    "$<BUILD_INTERFACE:${scs_SOURCE_DIR}/test;${scs_SOURCE_DIR}/include;${scs_SOURCE_DIR}/${LINSYS}>")

  add_test(NAME run_tests_indirect
    COMMAND run_tests_indirect
    WORKING_DIRECTORY ${scs_SOURCE_DIR}
    )
endif()
