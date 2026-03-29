
cmake_minimum_required(VERSION 3.19)
project(xxh3 LANGUAGES CXX)
if(TARGET ${PROJECT_NAME})
  message("The ${PROJECT_NAME} arledy included in main Project")
  return()
endif()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(xxh3 STATIC constexpr-xxh3.h)
set(PUBLIC_INCUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")

target_include_directories(${PROJECT_NAME} PUBLIC ${PUBLIC_INCUDE_DIR})
