project(cloudec C CXX)

cmake_minimum_required(VERSION 3.11)
cmake_policy(SET CMP0022 NEW)   # INTERFACE_LINK_LIBRARIES defines the link interface
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_OSX_ARCHITECTURES=arm64;x86_64)

message(STATUS "CMake Version: ${CMAKE_VERSION}")
message(STATUS "CMake Generator: ${CMAKE_GENERATOR}")
message(STATUS "CMake CXX Compiler ID: ${CMAKE_CXX_COMPILER_ID}")
message(STATUS "CMake CXX Compiler Version: ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "CMake System: ${CMAKE_SYSTEM}")
message(STATUS "CMake System Processor: ${CMAKE_SYSTEM_PROCESSOR}")
message(STATUS "CMake Module Path: ${CMAKE_MODULE_PATH}")


add_library(cloudec STATIC src/cloudec.cpp)

include(FetchContent)
FetchContent_Declare(
  Eigen
  GIT_REPOSITORY https://gitlab.com/libeigen/eigen.git
  GIT_TAG master
  GIT_SHALLOW TRUE
  GIT_PROGRESS TRUE)
set(EIGEN_BUILD_DOC OFF)
# note: To disable eigen tests,
# you should put this code in a add_subdirectory to avoid to change
# BUILD_TESTING for your own project too since variables are directory
# scoped
set(BUILD_TESTING OFF)
set(EIGEN_BUILD_PKGCONFIG OFF)
set( OFF)
FetchContent_MakeAvailable(Eigen)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake/modules)


find_package(FFTW)
if (FFTW_FOUND)
  message(STATUS "FFTW found")
  target_link_libraries(cloudec ${FFTW_LIBRARIES} Eigen3::Eigen)
else()
  message(STATUS "FFTW NOT found")
endif()

