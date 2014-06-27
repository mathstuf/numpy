cmake_minimum_required(VERSION 2.8.8)

project(numpy-static C)

message(STATUS "Building numpy using ${PYTHON_EXECUTABLE}...")
execute_process(
  COMMAND "${PYTHON_EXECUTABLE}" setup.py
          build
          --build-base "${CMAKE_CURRENT_BINARY_DIR}/numpy-build"
          install
          --prefix "${CMAKE_INSTALL_PREFIX}"
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
get_filename_component(config_path "${config_header}" PATH)
get_filename_component(config_path "${config_path}" PATH)
get_filename_component(core_dir "${config_path}" PATH)
set(core_src "${core_dir}/src")

set(CMAKE_C_FLAGS
  "-pthread -fno-strict-aliasing -DNDEBUG -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -D_GNU_SOURCE -fwrapv")

set(numpy_dir "${CMAKE_CURRENT_SOURCE_DIR}/numpy")
include_directories(
  "${numpy_dir}/core/src/private"
  "${numpy_dir}/core/src"
  "${numpy_dir}/core"
  "${numpy_dir}/core/src/npymath"
  "${numpy_dir}/core/src/multiarray"
  "${numpy_dir}/core/src/umath"
  "${numpy_dir}/core/src/npysort"
  "${numpy_dir}/core/include")
include_directories(SYSTEM "${PYTHON_INCLUDE_DIR}")
include_directories(
  "${config_path}/numpy"
  "${core_src}/multiarray"
  "${core_src}/umath")

add_library(numpy STATIC
  # multiarray module
  "${numpy_dir}/core/src/multiarray/multiarraymodule_onefile.c"

  # npymath library
  "${numpy_dir}/core/src/npymath/halffloat.c"
  "${core_src}/npymath/ieee754.c"
  "${core_src}/npymath/npy_math.c"
  "${core_src}/npymath/npy_math_complex.c"

  # _sort module
  "${core_src}/_sortmodule.c"

  # scalarmath module
  "${core_src}/scalarmathmodule.c"

  # umath module
  "${numpy_dir}/core/src/umath/umathmodule_onefile.c"

  # lapack_lite module
  "${numpy_dir}/linalg/lapack_litemodule.c"
  "${numpy_dir}/linalg/python_xerbla.c"

  # _dotblas module
  "${numpy_dir}/core/blasdot/_dotblas.c"

  # fftpack_lite module
  "${numpy_dir}/fft/fftpack.c"
  "${numpy_dir}/fft/fftpack_litemodule.c"

  # mtrand module
  "${numpy_dir}/random/mtrand/distributions.c"
  "${numpy_dir}/random/mtrand/mtrand.c"
  "${numpy_dir}/random/mtrand/initarray.c"
  "${numpy_dir}/random/mtrand/randomkit.c"

  # _compiled_base module
  "${numpy_dir}/lib/src/_compiled_base.c"

  # _capi module
  "${numpy_dir}/numarray/_capi.c"
  )
install(
  TARGETS     numpy
  DESTINATION lib
  COMPONENT   runtime)
