# SimSIMD library CMakeLists.txt
cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
project(
  simsimd
  VERSION 3.7.3
  LANGUAGES C CXX
  DESCRIPTION "Fastest SIMD-Accelerated Vector Similarity Functions for x86 and Arm"
  HOMEPAGE_URL "https://github.com/ashvardanian/simsimd")

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED YES)
set(CMAKE_C_EXTENSIONS NO)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED YES)
set(CMAKE_CXX_EXTENSIONS NO)

option(SIMSIMD_BUILD_BENCHMARKS "Compile a micro-benchmark for current ISA" OFF)

# Fetch external dependencies
include(FetchContent)

# Suppress building tests of Google Benchmark
set(BENCHMARK_ENABLE_TESTING OFF)
set(BENCHMARK_ENABLE_INSTALL OFF)
set(BENCHMARK_ENABLE_DOXYGEN OFF)
set(BENCHMARK_INSTALL_DOCS OFF)
set(BENCHMARK_DOWNLOAD_DEPENDENCIES ON)
set(BENCHMARK_ENABLE_GTEST_TESTS OFF)
set(BENCHMARK_USE_BUNDLED_GTEST ON)

FetchContent_Declare(
  benchmark
  GIT_REPOSITORY https://github.com/google/benchmark.git
  GIT_TAG v1.7.0)
FetchContent_MakeAvailable(benchmark)

# Default to Release build type if not set
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# Global compiler flags for debug and release
set(CMAKE_CXX_FLAGS_DEBUG "-g -fsanitize=address")
set(CMAKE_CXX_FLAGS_RELEASE "-O3")
set(CMAKE_C_FLAGS_DEBUG "-g -fsanitize=address")
set(CMAKE_C_FLAGS_RELEASE "-O3")

# Compiler-specific flags
if(CMAKE_CXX_COMPILER_ID MATCHES "^(Apple)?Clang$")
  if(NOT APPLE)
    add_compile_options(-march=native)
  endif()
  add_compile_options(-pedantic -ferror-limit=1)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  add_compile_options(-march=native -pedantic -fmax-errors=1 -Wno-tautological-constant-compare)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "IntelLLVM")
  add_compile_options(-w -ferror-limit=1)
endif()

# Define the header-only library
file(GLOB SIMSIMD_SOURCES include/simsimd/*.h)
add_library(simsimd INTERFACE)
target_sources(simsimd INTERFACE ${SIMSIMD_SOURCES})
target_include_directories(simsimd INTERFACE "${PROJECT_SOURCE_DIR}/include")

# Build benchmarks if required
if(SIMSIMD_BUILD_BENCHMARKS)
  find_package(Threads REQUIRED)
  add_executable(simsimd_bench cpp/bench.cxx)
  target_link_libraries(simsimd_bench simsimd Threads::Threads benchmark)
endif()
