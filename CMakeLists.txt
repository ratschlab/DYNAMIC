cmake_minimum_required(VERSION 2.6)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(USE_OPENMP "Enable multi-threading" OFF)

# Set a default build type if none was specified
if(NOT CMAKE_BUILD_TYPE)
  message(STATUS "Setting build type to 'Release' as none was specified.")
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
endif()

#set( CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/.. )

project (dynamic)

add_library(XXSDS_DYNAMIC INTERFACE)
target_compile_features(XXSDS_DYNAMIC INTERFACE cxx_std_11)
target_include_directories(XXSDS_DYNAMIC INTERFACE
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>"
    "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>")

if(TARGET tsl::hopscotch_map)
  target_link_libraries(XXSDS_DYNAMIC INTERFACE tsl::hopscotch_map)
else()
  include(ExternalProject)
  ExternalProject_Add(hopscotch_map
    GIT_REPOSITORY "https://github.com/Tessil/hopscotch-map.git"
    BUILD_IN_SOURCE TRUE
    UPDATE_COMMAND ""
    INSTALL_COMMAND ""
    BUILD_COMMAND ""
    CONFIGURE_COMMAND "")
  ExternalProject_Get_property(hopscotch_map SOURCE_DIR)
  target_include_directories(XXSDS_DYNAMIC INTERFACE "$<BUILD_INTERFACE:${SOURCE_DIR}/include>")
  add_dependencies(XXSDS_DYNAMIC hopscotch_map)
endif()

if(XXSDS_DYN_BUILD)
message("Building in ${CMAKE_BUILD_TYPE} mode")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --std=c++11")

if(XXSDS_DYN_MULTI_THREADED)
  find_package(OpenMP REQUIRED)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif(XXSDS_DYN_MULTI_THREADED)

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g3 -ggdb")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -ggdb -Ofast -fstrict-aliasing -DNDEBUG -march=native")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} -g -ggdb -Ofast -fstrict-aliasing -march=native")

add_executable(debug debug.cpp)
add_executable(rle_lz77_v1 rle_lz77_v1.cpp)
add_executable(rle_lz77_v2 rle_lz77_v2.cpp)
add_executable(h0_lz77 h0_lz77.cpp)
add_executable(rle_bwt rle_bwt.cpp)
add_executable(cw-bwt cw-bwt.cpp)
add_executable(benchmark benchmark.cpp)
add_executable(wm_string wm_string.cpp)

foreach(tool debug rle_lz77_v1 rle_lz77_v2 h0_lz77 rle_bwt cw-bwt benchmark wm_string)
  target_link_libraries(${tool} PRIVATE XXSDS_DYNAMIC)
endforeach()
endif()
