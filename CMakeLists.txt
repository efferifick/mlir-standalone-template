cmake_minimum_required(VERSION 3.20.0)

# CMP0116: Ninja generators transform `DEPFILE`s from `add_custom_command()`
# New in CMake 3.20. https://cmake.org/cmake/help/latest/policy/CMP0116.html
if(POLICY CMP0116)
  cmake_policy(SET CMP0116 OLD)
endif()

project(standalone-dialect LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 17 CACHE STRING "C++ standard to conform to")

find_package(MLIR REQUIRED CONFIG)

message(STATUS "Using MLIRConfig.cmake in: ${MLIR_DIR}")
message(STATUS "Using LLVMConfig.cmake in: ${LLVM_DIR}")

set(LLVM_RUNTIME_OUTPUT_INTDIR ${CMAKE_BINARY_DIR}/bin)
set(LLVM_LIBRARY_OUTPUT_INTDIR ${CMAKE_BINARY_DIR}/lib)
set(MLIR_BINARY_DIR ${CMAKE_BINARY_DIR})

list(APPEND CMAKE_MODULE_PATH "${MLIR_CMAKE_DIR}")
list(APPEND CMAKE_MODULE_PATH "${LLVM_CMAKE_DIR}")
include(TableGen)
include(AddLLVM)
include(AddMLIR)
include(HandleLLVMOptions)

include_directories(${LLVM_INCLUDE_DIRS})
include_directories(${MLIR_INCLUDE_DIRS})
include_directories(${PROJECT_SOURCE_DIR}/include)
include_directories(${PROJECT_BINARY_DIR}/include)
link_directories(${LLVM_BUILD_LIBRARY_DIR})
add_definitions(${LLVM_DEFINITIONS})

set(LLVM_LIT_ARGS "-sv" CACHE STRING "lit default options")
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/modules")
include(sanitizers)

add_subdirectory(include)
add_subdirectory(lib)
add_subdirectory(standalone-opt)

# Handle unittests when building out-of-tree against an installed version of
# LLVM/MLIR (not a build tree). Adapted from `llvm/flang/CMakeLists.txt`.
set(STANDALONE_GTEST_AVAILABLE 0)
if (TARGET llvm_gtest)
  # Installed gtest, via LLVM_INSTALL_GTEST.  Preferred.
  message(STATUS "LLVM GTest found, enabling unittests")
  set(STANDALONE_GTEST_AVAILABLE 1)
else()
    find_package(Threads REQUIRED)
    set(UNITTEST_DIR ${LLVM_THIRD_PARTY_DIR}/unittest)
    if (NOT EXISTS ${UNITTEST_DIR}/googletest/include/gtest/gtest.h)
      set(UNITTEST_DIR ${CMAKE_CURRENT_SOURCE_DIR}/llvm/third-party/unittest)
    endif()
    if (EXISTS ${UNITTEST_DIR}/googletest/include/gtest/gtest.h)
      add_llvm_library(llvm_gtest
        ${UNITTEST_DIR}/googletest/src/gtest-all.cc
        ${UNITTEST_DIR}/googlemock/src/gmock-all.cc
        LINK_COMPONENTS Support # llvm::raw_ostream
        BUILDTREE_ONLY
      )
      target_include_directories(llvm_gtest
        PUBLIC
        "${UNITTEST_DIR}/googletest/include"
        "${UNITTEST_DIR}/googlemock/include"
        PRIVATE
        "${UNITTEST_DIR}/googletest"
        "${UNITTEST_DIR}/googlemock"
      )
      target_link_libraries(llvm_gtest PUBLIC Threads::Threads)
      add_llvm_library(llvm_gtest_main
        ${UNITTEST_DIR}/UnitTestMain/TestMain.cpp
        LINK_LIBS llvm_gtest
        LINK_COMPONENTS Support # llvm::cl
        BUILDTREE_ONLY
      )
      set(STANDALONE_GTEST_AVAILABLE 1)
    else()
      message(WARNING "Skipping unittests since LLVM install does not include \
        gtest headers and libraries")
      set(STANDALONE_GTEST_AVAILABLE 0)
    endif()
endif()
if (STANDALONE_GTEST_AVAILABLE)
  add_subdirectory(unittests)
endif()

add_subdirectory(test)
