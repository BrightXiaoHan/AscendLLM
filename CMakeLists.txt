# CMake lowest version requirement
cmake_minimum_required(VERSION 3.5.1)

# project information
project(acl_execute_gemm)

# Compile options
add_compile_options(-std=c++11)

set(INC_PATH $ENV{DDK_PATH})

if (NOT DEFINED ENV{DDK_PATH})
    set(INC_PATH "/usr/local/Ascend")
    message(STATUS "set default INC_PATH: ${INC_PATH}")
else ()
    message(STATUS "env INC_PATH: ${INC_PATH}")
endif()

set(LIB_PATH $ENV{NPU_HOST_LIB})

# Dynamic libraries in the stub directory can only be used for compilation
if (NOT DEFINED ENV{NPU_HOST_LIB})
  set(LIB_PATH "${INC_PATH}/runtime/lib64/stub/")
    message(STATUS "set default LIB_PATH: ${LIB_PATH}")
else ()
    message(STATUS "env LIB_PATH: ${LIB_PATH}")
endif()

# Header path
include_directories(
    ${INC_PATH}/runtime/include/
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

# add host lib path
link_directories(
    ${LIB_PATH}
)

FILE(GLOB MAIN_SRC
    ${CMAKE_CURRENT_SOURCE_DIR}/csrc/*.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/csrc/*.h
)

add_library(ascendllm_runtime SHARED ${MAIN_SRC})
target_link_libraries(ascendllm_runtime PUBLIC
        ascendcl
        acl_cblas
        stdc++)

FILE(GLOB CLIENT_SRC
    ${CMAKE_CURRENT_SOURCE_DIR}/csrc/bin/*.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/csrc/bin/*.h
)

add_executable(ascendllm ${CLIENT_SRC})

target_link_libraries(ascendllm ascendllm_runtime)
