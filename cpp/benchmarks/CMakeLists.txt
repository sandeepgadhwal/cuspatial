#=============================================================================
# Copyright (c) 2019-2020, NVIDIA CORPORATION.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#=============================================================================

if(NOT CMAKE_CUDA_COMPILER)
  message(SEND_ERROR "CMake cannot locate a CUDA compiler")
endif()

###################################################################################################
# - common test utils -----------------------------------------------------------------------------

find_library(CUDFTESTUTIL_LIBRARY "libcudftestutil.a"
             HINTS "$ENV{CUDF_ROOT}/lib")

message(STATUS "CUDFTESTUTIL: CUDFTESTUTIL_LIBRARY set to ${CUDFTESTUTIL_LIBRARY}")

add_library(cudftestutil STATIC IMPORTED ${CUDFTESTUTIL_LIBRARY})
if (CUDFTESTUTIL_LIBRARY)
    set_target_properties(cudftestutil PROPERTIES IMPORTED_LOCATION ${CUDFTESTUTIL_LIBRARY})
endif (CUDFTESTUTIL_LIBRARY)

###################################################################################################
# - compiler function -----------------------------------------------------------------------------

function(ConfigureBench CMAKE_BENCH_NAME CMAKE_BENCH_SRC)
    add_executable(${CMAKE_BENCH_NAME}
                   ${CMAKE_BENCH_SRC}
                   "${CMAKE_CURRENT_SOURCE_DIR}/synchronization/synchronization.cpp")
    set_target_properties(${CMAKE_BENCH_NAME} PROPERTIES POSITION_INDEPENDENT_CODE ON)
    # By default the linker doesn't transitively add `DT_NEEDED` entries in executables like it does
    # for shared libraries, so to work around current libcudf build behavior we're adding the linker
    # flag
    target_link_libraries(${CMAKE_BENCH_NAME} "-Wl,--copy-dt-needed-entries")
    target_link_libraries(${CMAKE_BENCH_NAME} benchmark benchmark_main pthread cuspatial
                          cudftestutil)
    set_target_properties(${CMAKE_BENCH_NAME} PROPERTIES
                            RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/gbenchmarks")
endfunction(ConfigureBench)

###################################################################################################
# - include paths ---------------------------------------------------------------------------------

include_directories("${THRUST_INCLUDE_DIR}"
                    "${LIBCUDACXX_INCLUDE_DIR}")

include_directories("${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES}"
                    "${CMAKE_BINARY_DIR}/include"
                    "${CMAKE_SOURCE_DIR}/include"
                    "${CMAKE_SOURCE_DIR}"
                    "${CMAKE_SOURCE_DIR}/src"
                    "${GDAL_INCLUDE_DIRS}"
                    "${GTEST_INCLUDE_DIR}"
                    "${GBENCH_INCLUDE_DIR}"
                    "${RMM_INCLUDE}"
                    "${CUDF_INCLUDE}"
                    "${CMAKE_CURRENT_SOURCE_DIR}")

###################################################################################################
# - library paths ---------------------------------------------------------------------------------

link_directories("${CMAKE_CUDA_IMPLICIT_LINK_DIRECTORIES}" # CMAKE_CUDA_IMPLICIT_LINK_DIRECTORIES is an undocumented/unsupported variable containing the link directories for nvcc
                 "${CMAKE_BINARY_DIR}/lib"
                 "${CMAKE_BINARY_DIR}"
                 "${GDAL_LIBRARIES}"
                 "${CONDA_LINK_DIRS}"
                 "${GTEST_LIBRARY_DIR}"
                 "${GBENCH_LIBRARY_DIR}"
                 "${CUDF_LIBRARY}"
                 "${CUSPATIAL_LIBRARY}")

###################################################################################################
# - hausdorff benchmark ---------------------------------------------------------------------------

set(HAUSDORFF_BENCH_SRC
    "${CMAKE_CURRENT_SOURCE_DIR}/hausdorff_benchmark.cpp")

ConfigureBench(HAUSDORFF_BENCH "${HAUSDORFF_BENCH_SRC}")
