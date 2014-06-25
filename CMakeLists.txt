cmake_minimum_required(VERSION 2.8.8)

project(numpy-static C)

find_package(PythonInterp REQUIRED)
find_package(PythonLibs REQUIRED)

message(STATUS "Building numpy...")
execute_process(
  COMMAND "${PYTHON_EXECUTABLE}" setup.py
          build
          --build-base "${CMAKE_CURRENT_BINARY_DIR}/numpy-build"
  WORKING_DIRECTORY
          "${CMAKE_CURRENT_SOURCE_DIR}"
  RESULT_VARIABLE res
  OUTPUT_VARIABLE out
  ERROR_VARIABLE  err)
message(STATUS "Built numpy")

if (res)
  message(FATAL_ERROR
"Failed to build numpy:

  output:

    ${out}

  error:

    ${err}")
endif ()

file(GLOB config_header "${CMAKE_CURRENT_BINARY_DIR}/numpy-build/src.*/numpy/core/include/numpy/config.h")
get_filename_component(config_path "${config_header}" DIRECTORY)
get_filename_component(config_path "${config_path}" DIRECTORY)
get_filename_component(core_dir "${config_path}" DIRECTORY)
set(core_src "${core_dir}/src")

set(CMAKE_C_FLAGS
  "-pthread -fno-strict-aliasing -DNDEBUG -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -D_GNU_SOURCE -fwrapv")

set(numpy_dir "${CMAKE_CURRENT_SOURCE_DIR}/numpy/core")
set(numpy_src "${numpy_dir}/src")
include_directories(
  "${numpy_dir}/src/private"
  "${numpy_dir}/src"
  "${numpy_dir}"
  "${numpy_dir}/src/npymath"
  "${numpy_dir}/src/multiarray"
  "${numpy_dir}/src/umath"
  "${numpy_dir}/src/npysort"
  "${numpy_dir}/include")
include_directories(SYSTEM "${PYTHON_INCLUDE_DIR}")
include_directories(
  "${config_path}/numpy"
  "${core_src}/multiarray"
  "${core_src}/umath")

add_library(numpy STATIC
  # multiarray module
  "${numpy_src}/multiarray/multiarraymodule_onefile.c"

  # npymath library
  "${numpy_src}/npymath/halffloat.c"
  "${core_src}/npymath/ieee754.c"
  "${core_src}/npymath/npy_math.c"
  "${core_src}/npymath/npy_math_complex.c"

  # _sort module
  "${core_src}/_sortmodule.c"

  # scalarmath module
  "${core_src}/scalarmathmodule.c"

  # umath module
  "${numpy_src}/umath/umathmodule_onefile.c")
target_link_libraries(numpy
  "${PYTHON_LIBRARY}")
install(
  TARGETS     numpy
  DESTINATION lib
  COMPONENT   runtime)
