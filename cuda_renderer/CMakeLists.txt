cmake_minimum_required(VERSION 2.8)
set (CMAKE_CXX_STANDARD 14)
project(cuda_renderer)

option(USE_CUDA "use cuda or not" ON)

SET(CMAKE_BUILD_TYPE "Release")
#SET(CMAKE_BUILD_TYPE "Debug")

SET(CMAKE_CXX_FLAGS "-fopenmp -Wall -Wno-sign-compare -fPIC $ENV{CXXFLAGS}")
SET(CMAKE_CXX_FLAGS_DEBUG "-O0 -g2 -ggdb")
SET(CMAKE_CXX_FLAGS_RELEASE "-O3")

if(USE_CUDA)
set(CUDA_TOOLKIT_ROOT_DIR /usr/local/cuda-10.2)
add_definitions(-DCUDA_ON)
endif()
# opencv
find_package(OpenCV 3 REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})
list(APPEND renderer_inc ${OpenCV_INCLUDE_DIRS})
list(APPEND renderer_lib ${OpenCV_LIBS})

if(USE_CUDA)
# cuda
find_package(CUDA REQUIRED)
set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -std=c++11 -O3")
list(APPEND renderer_inc ${CUDA_INCLUDE_DIRS})
list(APPEND renderer_lib ${CUDA_LIBRARIES})
list(APPEND renderer_lib ${CUDA_CUBLAS_LIBRARIES})
endif()

# assimp, load ply
find_package(ASSIMP REQUIRED)
LINK_DIRECTORIES(/usr/local/lib)
list(APPEND renderer_inc ${ASSIMP_INCLUDE_DIRS})
list(APPEND renderer_lib ${ASSIMP_LIBRARIES})

find_package(Eigen3 REQUIRED)
include_directories(${EIGEN3_INCLUDE_DIR})

include_directories("include")
find_package(catkin REQUIRED 
    COMPONENTS 
        roscpp 
        OpenCV 
        fast_gicp
)
include_directories(${catkin_INCLUDE_DIRS})

# src
SET(renderer_cuda src/cuda/renderer.cu)
# SET(renderer_srcs src/renderer.cpp src/model.cpp)
SET(renderer_srcs src/model.cpp)
SET(renderer_knn src/cuda/knncuda.cu)

if(USE_CUDA)
CUDA_COMPILE(renderer_cuda_objs ${renderer_cuda} ${renderer_knn})
endif()
#cuda_add_library(renderer_cuda_objs
#     ${renderer_cuda} ${renderer_knn}
#  )

# Compile CUDA lib
# cuda_add_library(cuda_renderer_gpu
#     renderer.cu
#     knncuda.cu
#     # ${renderer_cuda} 
#     # ${renderer_knn}
# )
# target_include_directories(cuda_renderer_gpu PUBLIC
#     include
#     ${ASSIMP_INCLUDE_DIRS}
#     ${catkin_INCLUDE_DIRS}
#     ${EIGEN3_INCLUDE_DIR}
#     ${OpenCV_INCLUDE_DIRS}
#     ${CUDA_INCLUDE_DIRS}
# )
# target_link_libraries(cuda_renderer_gpu
#     ${CUDA_cublas_LIBRARY}
#     ${CUDA_LIBRARIES}
#     ${catkin_LIBRARIES}
# )
# 
catkin_package(
    CATKIN_DEPENDS
        fast_gicp
    INCLUDE_DIRS 
        include
    LIBRARIES 
        ${PROJECT_NAME}
)
# lib & test exe
add_library(cuda_renderer
               ${renderer_srcs}
               ${renderer_cuda} 
               ${renderer_knn}
               ${renderer_cuda_objs} 
)
target_include_directories(cuda_renderer PUBLIC ${renderer_inc} ${catkin_INCLUDE_DIRS})
target_link_libraries(cuda_renderer PUBLIC ${renderer_lib} ${catkin_LIBRARIES} )


#add_executable(renderer_test test.cpp)
#target_link_libraries(renderer_test cuda_renderer)


install(DIRECTORY include/${PROJECT_NAME}/
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})

