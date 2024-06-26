﻿# CMakeLists.txt: CMake project for simple_dll, include source and define
# project specific logic here.
cmake_minimum_required(VERSION 3.8)

# Enable Hot Reload for MSVC compilers if supported.
if(POLICY CMP0141)
    cmake_policy(SET CMP0141 NEW)
    set(CMAKE_MSVC_DEBUG_INFORMATION_FORMAT "$<IF:$<AND:$<C_COMPILER_ID:MSVC>,$<CXX_COMPILER_ID:MSVC>>,$<$<CONFIG:Debug,RelWithDebInfo>:EditAndContinue>,$<$<CONFIG:Debug,RelWithDebInfo>:ProgramDatabase>>")
endif()

set(TARGET_NAME "rcvpose")
project(${TARGET_NAME})

set(CMAKE_PREFIX_PATH "/home/jadoo/libtorch/share/cmake")
#set(TORCH_DIR "/home/jadoo/libtorch")
#set(TORCH_LIBRARY "/home/jadoo/libtorch/lib")
#set(Torch required PATHS "/home/jadoo/libtorch")
set(OpenCV_DIR /usr/local/lib/cmake/opencv4)

# Must fix on pc
 set(CUDA_TOOLKIT_ROOT_DIR "/usr/local/cuda-12.1")
# 
 set(CUDA_HOME "/usr/local/cuda-12.1")
# 
 set (CUDA_NVCC_EXECUTABLE "/usr/local/cuda-12.1/bin/nvcc")
# 
 set(CMAKE_CUDA_COMPILER "/usr/local/cuda-12.1/bin/nvcc")
#

find_package(OpenMP)

find_package(Boost REQUIRED COMPONENTS filesystem)
include_directories(${Boost_INCLUDE_DIRS})

find_package(OpenCV REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS} )

MESSAGE( STATUS "OpenCV_INCLUDE_DIRS: " ${OpenCV_INCLUDE_DIRS} )
MESSAGE( STATUS "OpenCV_LIBRARIES: " ${OpenCV_LIBRARIES} )
MESSAGE( STATUS "OpenCV_LIBRARY_DIRS: " ${OpenCV_LIBRARY_DIRS} )
MESSAGE( STATUS "OpenCV_VERSION: " ${OpenCV_VERSION} )

find_package(Open3D REQUIRED)
include_directories(${Open3D_INCLUDE_DIRS} "/usr/local/include/")

find_package(Torch REQUIRED)
include_directories(${TORCH_INCLUDE_DIRS})

find_package(Eigen3 REQUIRED)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")

include(FetchContent)

FetchContent_Declare(
  json
  GIT_REPOSITORY https://github.com/nlohmann/json.git
  GIT_TAG v3.10.5 # Adjust the tag to the version you want to use
)

# Make the fetched content available.
FetchContent_MakeAvailable(json)

# Add source files to this project's target.
set(SOURCE_FILES
    "rcvpose.cpp"
    "rcvpose.h"
    "data_loader.cpp"
    "data_loader.h"
    "RMapDataset.cpp"
    "RMapDataset.h"
    "trainer.h"
    "trainer.cpp"
    "models/bottleneck.h"
    "models/bottleneck.cpp"
    "models/denseFCNResNet152.h"
    "models/denseFCNResNet152.cpp"
    "models/resFCNResNet152.h"
    "models/resFCNResNet152.cpp"
    "JACOBI.h"
    "JACOBI.c"
    "lmshorn.c"
    "lmshorn.h"
    "accumulator_space.h"
    "accumulator_space.cpp"
    "npy.hpp"
    "AccSpaceMath.h" 
    "AccSpaceMath.cpp"
    "AccSpaceIO.h" 
    "AccSpaceIO.cpp"
    "ransac.h"
    "ransac.cpp"
)

option(BUILD_DLL "Build DLL" OFF)
# Conditionally include the test.cpp file based on BUILD_DLL variable.
if(BUILD_DLL)
    list(REMOVE_ITEM SOURCE_FILES "test.cpp")
    add_library (${TARGET_NAME} SHARED ${SOURCE_FILES})
else()
	list(APPEND SOURCE_FILES "test.cpp")
    add_executable(${TARGET_NAME} ${SOURCE_FILES})
endif()

if(OpenMP_CXX_FOUND)
    target_link_libraries(${TARGET_NAME} PUBLIC OpenMP::OpenMP_CXX)
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE ${TORCH_LIBRARIES} ${OpenCV_LIBRARIES} ${Open3D_LIBRARIES} ${Boost_LIBRARIES} Eigen3::Eigen -lstdc++fs nlohmann_json::nlohmann_json)


if(MSVC)
    file(GLOB TORCH_DLLS "${TORCH_INSTALL_PREFIX}/lib/*.dll")
    add_custom_command(TARGET ${TARGET_NAME}
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${TORCH_DLLS}
        $<TARGET_FILE_DIR:${TARGET_NAME}>
    )
endif(MSVC)

if(CMAKE_VERSION VERSION_GREATER 3.12)
    set_property(TARGET ${TARGET_NAME} PROPERTY CXX_STANDARD 20)
endif()

# TODO: Add tests and install targets if needed.
