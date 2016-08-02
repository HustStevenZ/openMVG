#include_directories(${CMAKE_SOURCE_DIR}/include algorithms util nn .)

add_definitions(-D_FLANN_VERSION=${FLANN_VERSION})


configure_file(${CMAKE_CURRENT_SOURCE_DIR}/flann/config.h.in ${CMAKE_CURRENT_SOURCE_DIR}/flann/config.h)
        
file(GLOB_RECURSE C_SOURCES flann.cpp lz4.c lz4hc.c)
file(GLOB_RECURSE CPP_SOURCES flann_cpp.cpp lz4.c lz4hc.c)
file(GLOB_RECURSE CU_SOURCES *.cu)

add_library(flann_cpp_s STATIC ${CPP_SOURCES})
if(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_CLANG)
    set_target_properties(flann_cpp_s PROPERTIES COMPILE_FLAGS -fPIC)
endif()
SET_PROPERTY(TARGET flann_cpp_s PROPERTY COMPILE_DEFINITIONS FLANN_STATIC FLANN_USE_CUDA)

if (BUILD_CUDA_LIB)
    SET(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS};-DFLANN_USE_CUDA")
    if(CMAKE_COMPILER_IS_GNUCC)
                set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS};-Xcompiler;-fPIC" )
        if (NVCC_COMPILER_BINDIR)
            set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS};--compiler-bindir=${NVCC_COMPILER_BINDIR}")
        endif()
    else()
            set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS};" )
    endif()
    cuda_add_library(flann_cuda_s STATIC ${CU_SOURCES})
    set_property(TARGET flann_cuda_s PROPERTY COMPILE_DEFINITIONS FLANN_STATIC FLANN_USE_CUDA)
endif()

INSTALL(
  TARGETS flann_cpp_s
  DESTINATION lib
  EXPORT openMVG-targets)


INSTALL(
  TARGETS flann_cuda_s
  DESTINATION lib
  EXPORT openMVG-targets)


INSTALL (
  DIRECTORY .
  DESTINATION ${FLANN_INCLUDE_INSTALL_DIR}
  FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp"
)
#if(CMAKE_SYSTEM_NAME STREQUAL "Linux" AND CMAKE_COMPILER_IS_GNUCC)
#    add_library(flann_cpp SHARED "")
#    set_target_properties(flann_cpp PROPERTIES LINKER_LANGUAGE CXX)
#    target_link_libraries(flann_cpp -Wl,-whole-archive flann_cpp_s -Wl,-no-whole-archive)
#else()
#    add_library(flann_cpp SHARED ${CPP_SOURCES})
#endif()

#set_target_properties(flann_cpp PROPERTIES
#   VERSION ${FLANN_VERSION}
#   SOVERSION ${FLANN_SOVERSION}
#   DEFINE_SYMBOL FLANN_EXPORTS
#) 

