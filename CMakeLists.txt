
cmake_minimum_required(VERSION 3.19)
project(xxh3 LANGUAGES CXX)
if(TARGET ${PROJECT_NAME})
  message("The ${PROJECT_NAME} already included in main Project")
  return()
endif()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# constexpr-xxh3 is a header-only library (single header, no sources),
# so it is declared as an INTERFACE library.
add_library(xxh3 INTERFACE)
set(PUBLIC_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")

target_include_directories(${PROJECT_NAME} INTERFACE ${PUBLIC_INCLUDE_DIR})
