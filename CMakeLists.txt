# EVMC: Ethereum Client-VM Connector API.
# Copyright 2016-2019 The EVMC Authors.
# Licensed under the Apache License, Version 2.0.

cmake_minimum_required(VERSION 3.5)

if(TARGET evmc)
    # The evmc library has been already created (probably by other submodule).
    return()
endif()

option(EVMC_TESTING "Build EVMC examples, tests and test tools (i.e. everything)" OFF)
option(EVMC_TEST_TOOLS "Build EVMC test tools" ${EVMC_TESTING})
if(EVMC_TESTING OR EVMC_TEST_TOOLS)
    set(hunter_and_examples_required TRUE)
endif()
option(EVMC_EXAMPLES "Build EVMC examples" ${hunter_and_examples_required})
option(HUNTER_ENABLED "Enable Hunter package manager support" ${hunter_and_examples_required})

include(cmake/cable/bootstrap.cmake)
include(CableBuildType)
include(CableCompilerSettings)
include(CableToolchains)
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

if(HUNTER_ENABLED)
    set(HUNTER_CONFIGURATION_TYPES Release CACHE STRING "Build type of Hunter packages")
    include(HunterGate)
    include(Hunter/init)
    include(defaults/HunterCacheServers)
endif()

cable_configure_toolchain(DEFAULT cxx11-pic)
cable_set_build_type(DEFAULT Release CONFIGURATION_TYPES Debug Release)

project(evmc)
set(PROJECT_VERSION 6.3.0-dev)

cable_configure_compiler(NO_STACK_PROTECTION)

set(include_dir ${PROJECT_SOURCE_DIR}/include)

add_library(evmc INTERFACE)
add_library(evmc::evmc ALIAS evmc)
target_include_directories(evmc INTERFACE $<BUILD_INTERFACE:${include_dir}>$<INSTALL_INTERFACE:include>)
install(TARGETS evmc EXPORT evmcTargets)

install(DIRECTORY include/evmc DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

add_subdirectory(lib)

if(EVMC_TESTING)
    enable_testing()
endif()

if(EVMC_TESTING OR EVMC_TEST_TOOLS)
    add_subdirectory(test)
endif()

if(EVMC_TESTING OR EVMC_EXAMPLES)
    add_subdirectory(examples)
endif()

write_basic_package_version_file(evmcConfigVersion.cmake COMPATIBILITY ExactVersion)

configure_package_config_file(
    cmake/Config.cmake.in
    evmcConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/evmc
)

install(
    EXPORT evmcTargets
    NAMESPACE evmc::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/evmc
)
install(
    FILES
    ${CMAKE_CURRENT_BINARY_DIR}/evmcConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/evmcConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/evmc
)

if(WIN32)
    set(CPACK_GENERATOR ZIP)
else()
    set(CPACK_GENERATOR TGZ)
endif()
string(TOLOWER ${CMAKE_SYSTEM_NAME} system_name)
set(CPACK_PACKAGE_FILE_NAME ${PROJECT_NAME}-${PROJECT_VERSION}-${system_name})
set(CPACK_PACKAGE_CHECKSUM SHA256)
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
include(CPack)
