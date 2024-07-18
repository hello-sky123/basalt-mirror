cmake_minimum_required(VERSION 3.10...3.18) # 指定了cmake的版本范围

project(basalt)
# 将当前项目源代码目录下的cmake_modules目录添加到CMAKE_MODULE_PATH变量的开头，并保留之前的搜索路径
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules/" ${CMAKE_MODULE_PATH})

# 检查有没有设置EIGEN_ROOT变量，如果没有设置，则将其设置为thirdparty/basalt-headers/thirdparty/eigen目录
if(NOT EIGEN_ROOT)
  set(EIGEN_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/basalt-headers/thirdparty/eigen")
endif()

string(TOLOWER "${PROJECT_NAME}" PROJECT_NAME_LOWERCASE) # 将PROJECT_NAME转换为小写，并存储在PROJECT_NAME_LOWERCASE变量中
# 在系统中查找dpkg程序的可执行文件路径，并将结果存储在DPKG_PROGRAM变量中，DOC是一个可选参数，用于提供关于查找的程序的简短描述
find_program(DPKG_PROGRAM dpkg DOC "dpkg program of Debian-based systems")
if(DPKG_PROGRAM)
  # execute_process用于执行外部进程或命令
  execute_process(
    COMMAND ${DPKG_PROGRAM} --print-architecture # 指定要执行的命令，--print-architecture是dpkg的一个选项，用于输出系统的架构类型（如amd64、arm64等）
    OUTPUT_VARIABLE CPACK_DEBIAN_PACKAGE_ARCHITECTURE # 指定将命令的输出存储在变量CPACK_DEBIAN_PACKAGE_ARCHITECTURE中
    OUTPUT_STRIP_TRAILING_WHITESPACE) # 这个选项用于去除命令输出结果中的尾部空白字符
endif(DPKG_PROGRAM)

# lsb_release命令可以获取系统的发行版信息
find_program(LSB_RELEASE_PROGRAM lsb_release DOC "lsb_release program of Debian-based systems")
if(LSB_RELEASE_PROGRAM)
  execute_process(COMMAND ${LSB_RELEASE_PROGRAM} -rs # 获取系统的发行版本号
    OUTPUT_VARIABLE LSB_RELEASE_ID_SHORT
    OUTPUT_STRIP_TRAILING_WHITESPACE)

  # 根据发行版本，设置不同的依赖项
  if(${LSB_RELEASE_ID_SHORT} EQUAL "24.04")
    set(DEBIAN_DEPENDS "libtbb12, liblz4-1, libbz2-1.0, libboost-filesystem1.83.0, libboost-date-time1.83.0, libboost-program-options1.83.0, libboost-regex1.83.0, libopencv-dev, libglew2.2, libjpeg8, libpng16-16, librealsense2, librealsense2-dkms, librealsense2-gl, librealsense2-utils")

  elseif(${LSB_RELEASE_ID_SHORT} EQUAL "22.04")
    set(DEBIAN_DEPENDS "libtbb12, liblz4-1, libbz2-1.0, libboost-filesystem1.74.0, libboost-date-time1.74.0, libboost-program-options1.74.0, libboost-regex1.74.0, libopencv-dev, libglew2.2, libjpeg8, libpng16-16, librealsense2, librealsense2-dkms, librealsense2-gl, librealsense2-utils")

  elseif(${LSB_RELEASE_ID_SHORT} EQUAL "20.04")
    set(DEBIAN_DEPENDS "libtbb2, liblz4-1, libbz2-1.0, libboost-filesystem1.71.0, libboost-date-time1.71.0, libboost-program-options1.71.0, libboost-regex1.71.0, libopencv-dev, libglew2.1, libjpeg8, libpng16-16, librealsense2, librealsense2-dkms, librealsense2-gl, librealsense2-utils")

  elseif(${LSB_RELEASE_ID_SHORT} EQUAL "18.04")
    set(DEBIAN_DEPENDS "libtbb2, liblz4-1, libbz2-1.0, libboost-filesystem1.65.1, libboost-date-time1.65.1, libboost-program-options1.65.1, libboost-regex1.65.1, libopencv-dev, libglew2.0, libjpeg8, libpng16-16, librealsense2, librealsense2-dkms, librealsense2-gl, librealsense2-utils")

  elseif(${LSB_RELEASE_ID_SHORT} EQUAL "16.04")
    set(DEBIAN_DEPENDS "libtbb2, liblz4-1, libbz2-1.0, libboost-filesystem1.58.0, libboost-date-time1.58.0, libboost-program-options1.58.0, libboost-regex1.58.0, libopencv-dev, libglew1.13, libjpeg8, libpng12-0, libstdc++6, librealsense2, librealsense2-dkms, librealsense2-gl, librealsense2-utils")
  endif()

endif(LSB_RELEASE_PROGRAM)

string(TIMESTAMP PROJECT_VERSION_REVISION "%Y%m%d%H%M") # 获取当前的日期和时间，然后将结果存储在PROJECT_VERSION_REVISION变量中

# 配置Debian软件包的相关信息
set(CPACK_GENERATOR "DEB")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Vladyslav Usenko <vlad.usenko@tum.de>")
set(CPACK_PACKAGE_VERSION_MAJOR "0")
set(CPACK_PACKAGE_VERSION_MINOR "1")
set(CPACK_PACKAGE_VERSION_PATCH "0-${PROJECT_VERSION_REVISION}~${LSB_RELEASE_ID_SHORT}")
set(CPACK_DEBIAN_PACKAGE_DEPENDS ${DEBIAN_DEPENDS})
set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME_LOWERCASE}_${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE}")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
include(CPack)

# Configure CCache if available. ccache是编译器加速工具，可以加速C/C++代码的编译速度
# 编译器启动器（例如ccache）可以通过在编译过程中缓存编译结果，从而加快编译速度
if(NOT CMAKE_C_COMPILER_LAUNCHER AND NOT CMAKE_CXX_COMPILER_LAUNCHER)
  find_program(CCACHE_PROGRAM ccache)
  if(CCACHE_PROGRAM)
    message(STATUS "Found ccache: ${CCACHE_PROGRAM}")
    set(CMAKE_C_COMPILER_LAUNCHER   ${CCACHE_PROGRAM})
    set(CMAKE_CXX_COMPILER_LAUNCHER ${CCACHE_PROGRAM})
  else()
    message(STATUS "Dind't find ccache")
  endif()
else()
  message(STATUS "Compiler launcher already set. Not configuring ccache.")
  message(STATUS "CMAKE_C_COMPILER_LAUNCHER: ${CMAKE_C_COMPILER_LAUNCHER}")
  message(STATUS "CMAKE_CXX_COMPILER_LAUNCHER: ${CMAKE_CXX_COMPILER_LAUNCHER}")
endif()

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# 编译器选项-march用于指定目标架构（architecture），即编译器应为哪个CPU架构生成优化代码。
# 设置为native可以使编译器自动检测并使用当前编译机器的特性进行优化，从而生成更高效的代码
if(NOT CXX_MARCH)
  set(CXX_MARCH native)
endif()

# 如果不是Apple系统或者不是arm64架构，则设置BASALT_MARCH_FLAGS标志
if(NOT APPLE OR NOT CMAKE_SYSTEM_PROCESSOR STREQUAL "arm64")
  set(BASALT_MARCH_FLAGS "-march=${CXX_MARCH}")
else()
  message(STATUS "Running on Apple ${CMAKE_SYSTEM_PROCESSOR}. Disabled -march flag.")
endif()

set(CMAKE_CXX_STANDARD 17)
# 强制要求使用指定的C++标准。如果设置为ON，编译器必须支持所指定的C++标准（在这里是C++17），否则编译将失败
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF) # 禁用编译器特定的C++扩展
# 生成位置无关代码（Position Independent Code，PIC）。这对于创建共享库（shared libraries）是必需的，因为位置无关代码可以在内存中的任何位置运行，
# 而不会依赖于硬编码的内存地址。这有助于提高代码的安全性和灵活性，尤其是在动态加载库时。
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# Flags used for CHECK_CXX_SOURCE_COMPILES
# CMAKE_REQUIRED_FLAGS：这是一个CMake变量，用于指定在配置测试期间（如编译器功能检测和库检测）使用的编译标志。
# "-Wno-error"：这是一个编译器标志，用于告诉编译器将所有警告视为警告，而不是错误。
set(CMAKE_REQUIRED_FLAGS "-Wno-error")

# save flags passed by user
set(BASALT_PASSED_CXX_FLAGS "${CMAKE_CXX_FLAGS}") # 保存当前的C++编译标志
# -O0：关闭所有优化，以便更容易调试。-g：生成调试信息，将Eigen库中的矩阵初始化为NaN，以便更容易发现未初始化的矩阵
set(CMAKE_CXX_FLAGS_DEBUG  "-O0 -g -DEIGEN_INITIALIZE_MATRICES_BY_NAN")          # cmake default: "-g"
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O3 -g -DEIGEN_INITIALIZE_MATRICES_BY_NAN")  # cmake default: "-O2 -g -DNDEBUG"
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG") # 禁用断言和调试信息                    # cmake default: "-O3 -DNDEBUG"
set(CMAKE_CXX_FLAGS_CIDEBUG  "-O0 -DEIGEN_INITIALIZE_MATRICES_BY_NAN")          # CI version with no debug symbols
set(CMAKE_CXX_FLAGS_CIRELWITHDEBINFO "-O3 -DEIGEN_INITIALIZE_MATRICES_BY_NAN")  # CI version with no debug symbols

# base set of compile flags
# -Wall：启用所有警告，-Wextra：启用一些额外的警告，-Werror：将所有警告视为错误，-Wno-error=unused-parameter：禁用未使用参数的警告，
# -ftemplate-backtrace-limit=0：禁用模板回溯限制，置为0表示不限制回溯深度。对于复杂的模板代码，编译器在生成错误信息时可能会回溯多层模板嵌套，设置这个选项可以帮助调试模板相关的错误。
set(BASALT_CXX_FLAGS "-Wall -Wextra -Werror -Wno-error=unused-parameter -ftemplate-backtrace-limit=0")

# clang-specific compile flags
if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR CMAKE_CXX_COMPILER_ID STREQUAL "AppleClang")
  set(BASALT_CXX_FLAGS "${BASALT_CXX_FLAGS} -Wno-exceptions -fcolor-diagnostics -frelaxed-template-template-args -Wno-error=deprecated-declarations")

  if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 10)
      # These are disabled to avoid lot's of warnings in Eigen code with clang 10
      set(BASALT_CXX_FLAGS "${BASALT_CXX_FLAGS} -Wno-misleading-indentation -Wno-error=deprecated-copy")
  endif()

  if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 15)
      # These are disabled to avoid lot's of warnings in Eigen code with clang 15
      set(BASALT_CXX_FLAGS "${BASALT_CXX_FLAGS} -Wno-error=unused-but-set-variable")
  endif()

  #   - Added TBB_USE_GLIBCXX_VERSION macro to specify the version of GNU
  #     libstdc++ when it cannot be properly recognized, e.g. when used
  #     with Clang on Linux* OS. Adopted from https://github.com/wjakob/tbb
  if(NOT TBB_USE_GLIBCXX_VERSION AND UNIX AND NOT APPLE)
    if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
      string(REPLACE "." "0" TBB_USE_GLIBCXX_VERSION ${CMAKE_CXX_COMPILER_VERSION})
    endif()
    add_definitions(-DTBB_USE_GLIBCXX_VERSION=${TBB_USE_GLIBCXX_VERSION})
  endif()
else()
  # -Wno-error=maybe-uninitialized： 这个编译标志告诉编译器，将变量可能在使用前没有被初始化警告降级为普通警告，而不是错误。
  # -Wno-error=implicit-fallthrough： 这个编译标志告诉编译器，将在switch语句中表示某个case分支没有break、return等结束语句，
  # 直接继续执行下一个case分支的代码警告降级为普通警告，而不是错误。
  set(BASALT_CXX_FLAGS "${BASALT_CXX_FLAGS} -Wno-error=maybe-uninitialized -Wno-error=implicit-fallthrough")

  if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 9)
      # These are disabled to avoid lot's of warnings in Eigen code with gcc-9
      # 将代码中使用了被标记为弃用的拷贝构造函数的警告降级为普通警告。GCC9及更高版本对这种情况进行了更严格的检查和警告
      set(BASALT_CXX_FLAGS "${BASALT_CXX_FLAGS} -Wno-error=deprecated-copy")
  endif()
  if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 10.0)
    # These are disabled due to warnings in pangolin
    #set(BASALT_CXX_FLAGS "${BASALT_CXX_FLAGS} -Wno-error=parentheses")
    set(BASALT_CXX_FLAGS "${BASALT_CXX_FLAGS} -Wno-parentheses") # 代码中可能存在需要括号来消除歧义的地方
  endif()
endif()


# Set platform / compiler specific compile flags and checks
if(APPLE)
  # Need to investigate how to reliably detect and use OpenMP on macOS...
#  set(USE_OPENMP_DEFAULT OFF)

  # Among others, setting CMAKE_FIND_FRAMEWORK to LAST fixed issues
  # with installed Mono that contains old headers (libpng, ...).
  # See: https://github.com/openMVG/openMVG/issues/1349#issuecomment-401492811
  set(CMAKE_FIND_FRAMEWORK LAST)

  if(CMAKE_SYSTEM_VERSION VERSION_LESS 19.0.0)
      # use brewed llvm's libc++
      include_directories("/usr/local/opt/llvm/include/c++/v1")
      link_directories("/usr/local/opt/llvm/lib")
      add_compile_options("-nostdinc++")
      #set(STD_CXX_FS c++fs)

      # Workaround for cmake not to filter the manually added standard include path
      # See: https://gitlab.kitware.com/cmake/cmake/issues/19227#note_669894
      list(REMOVE_ITEM CMAKE_CXX_IMPLICIT_INCLUDE_DIRECTORIES "/usr/local/opt/llvm/include/c++/v1")
  endif()

  if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    message(STATUS "Detected macOS with non-Apple clang")

  elseif(CMAKE_CXX_COMPILER_ID STREQUAL "AppleClang")
    message(STATUS "Detected macOS with Apple clang")
    # Apple clang on macOS < 10.14 Mojave is too old
    if(CMAKE_SYSTEM_VERSION VERSION_LESS 18.0.0)
      message(WARNING "Detected Darwin version ${CMAKE_SYSTEM_VERSION}, which is earlier than macos 10.14 Mojave. Apple clang is too old and not supported. Use clang from homebrew.")
    endif()

  else()
    message(WARNING "Detected macOS with unsupported compiler ${CMAKE_CXX_COMPILER_ID}")
  endif()

elseif(UNIX)
#  set(USE_OPENMP_DEFAULT ON)

  # assume libstdc++，在C++17标准中，引入了新的文件系统库，位于<filesystem>头文件中。然而，
  # 在某些编译器（例如 GCC7和8）中，文件系统库并不默认链接到标准库，需要显式地链接stdc++fs库
  set(STD_CXX_FS stdc++fs)

  if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    message(STATUS "Detected Linux with clang.")

    # Note: Whatever the issue was, this seems to not be an issue any more, at least with more recent clang and Ubuntu versions (tested clang-12 and Ubuntu 18.04)
    #message(WARNING "Clang on Linux is currently not fully supported. You'll likely need to get a recent version of TBB.")
    #set(LINUX_CLANG 1)

  elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    message(STATUS "Detected Linux with gcc.")

  else()
    message(WARNING "Detected Linux with unsupported compiler ${CMAKE_CXX_COMPILER_ID}")
  endif()

else()
  message(WARNING "Only Linux and macOS are currently supported")
endif()


# OpenMP option and compile flags
#
# Note: OpenMP and TBB don't mix well, so we disable Eigen's parallelization.
# It's trying to paralellize matrix products during SC, which we run in a parallel_reduce using TBB.
# Turns out using OpenMP can slow down the computby factor 10-100x! So for now we discable it completely.
# One place where Eigen's parallelization could still have been useful is the CG solver in the mapper.
# We could in the future investiagte other implementations (paralellized with TBB) or selectively enabling
# Eigen's parallelization just for CG, setting number of threads to 1 everywhere else.
# Another way to ensure Eigen doesn't use OpenMP regardless of how it was built is setting the environment
# variable OMP_NUM_THREADS=1 beofre running the application.
#
# See: https://eigen.tuxfamily.org/dox/TopicMultiThreading.html
#
# If we enable BLAS / LAPACK either directly or via thirdparty libs like ceres,
# make sure to disable OpenMP for the linked BLAS library. In particular on Ubuntu it seems OpenBLAS is often installed,
# and it can have similar issues in multithreaded applications if it's own parallelization with OpenMP is enabled.
# You can set the environment varaibles OPENBLAS_NUM_THREADS=1 or OMP_NUM_THREADS=1. This is also mentioned in the ceres
# installation documentation.
#
# See also: https://github.com/xianyi/OpenBLAS/wiki/faq#multi-threaded
#
# Set EIGEN_DONT_PARALLELIZE to be sure it doesn't use openmp,
# just in case some dependency enables openmp without us noticing.
set(BASALT_CXX_FLAGS "${BASALT_CXX_FLAGS} -DEIGEN_DONT_PARALLELIZE")

#option(USE_OPENMP "Use OpenMP (e.g. for parallel computation in Eigen)" ${USE_OPENMP_DEFAULT})
#if(USE_OPENMP)
#  message(STATUS "OpenMP Enabled")
#  set(BASALT_CXX_FLAGS "${BASALT_CXX_FLAGS} -fopenmp")
#else()
#  message(STATUS "OpenMP Disabled")
#endif()

# Make specialization for double / float optional.
# Disabling them can be useful for better compile times during development.
# option定义一个布尔变量，用于控制是否启用某些功能。
option(BASALT_INSTANTIATIONS_DOUBLE "Instatiate templates for Scalar=double." ON)
option(BASALT_INSTANTIATIONS_FLOAT "Instatiate templates for Scalar=float." ON)

# list(APPEND ...)：CMake命令，用于向列表中追加元素
if(BASALT_INSTANTIATIONS_DOUBLE)
  list(APPEND BASALT_COMPILE_DEFINITIONS BASALT_INSTANTIATIONS_DOUBLE)
endif()
if(BASALT_INSTANTIATIONS_FLOAT)
  list(APPEND BASALT_COMPILE_DEFINITIONS BASALT_INSTANTIATIONS_FLOAT)
endif()

# setup combined compiler flags
set(CMAKE_CXX_FLAGS "${BASALT_CXX_FLAGS} ${BASALT_MARCH_FLAGS} ${BASALT_PASSED_CXX_FLAGS}")

set(EIGEN_INCLUDE_DIR_HINTS ${EIGEN_ROOT}) # 提供查找Eigen包时的包含目录提示
find_package(Eigen3 3.4.0 EXACT REQUIRED MODULE) # 查找Eigen3包，指定版本为3.4.0，并且要求模块方式查找
message(STATUS "Found Eigen headers in: ${EIGEN3_INCLUDE_DIR}")
if(NOT EIGEN3_INCLUDE_DIR MATCHES "^${EIGEN_ROOT}")
  message(WARNING "Found Eigen headers are outside of specified EIGEN_ROOT '${EIGEN_ROOT}'")
endif()

find_package(TBB REQUIRED)
message(STATUS "Found TBB ${TBB_VERSION_MAJOR}.${TBB_VERSION_MINOR} (interface version ${TBB_INTERFACE_VERSION}) headers in: ${TBB_INCLUDE_DIRS}")
if (TBB_INTERFACE_VERSION LESS 11004)
  # enable global_control header for earlier TBB versions (Ubuntu 16.04, 18.04)
  add_definitions(-DTBB_PREVIEW_GLOBAL_CONTROL) # 预处理器定义，用于启用TBB的global_control特性。
endif()

# NOTE: not specifying version, since 2, 3 or 4 is fine
find_package(OpenCV REQUIRED COMPONENTS core imgproc calib3d highgui)
message(STATUS "Found OpenCV ${OpenCV_VERSION} headers in: ${OpenCV_INCLUDE_DIRS}")
message(STATUS "Found OpenCV_LIBS: ${OpenCV_LIBS}")

# Add our own custom scoped opencv target since none is provided by OpenCV itself
# 创建一个名为basalt::opencv的接口库，并为其设置包含目录和链接库属性，指定库类型为接口库。接口库不包含实现，
# 只包含接口定义，通常用于提供编译器选项、宏定义、头文件路径等。IMPORTED：指定库是导入的，即该库不是由当前项目生成的，而是从外部引入
add_library(basalt::opencv INTERFACE IMPORTED)
# 设置目标的属性，INTERFACE_INCLUDE_DIRECTORIES：指定接口库的包含目录，INTERFACE_LINK_LIBRARIES：指定接口库的链接库
set_property(TARGET basalt::opencv PROPERTY INTERFACE_INCLUDE_DIRECTORIES ${OpenCV_INCLUDE_DIRS})
set_property(TARGET basalt::opencv PROPERTY INTERFACE_LINK_LIBRARIES ${OpenCV_LIBS})

find_package(fmt REQUIRED)
message(STATUS "Found {fmt} ${fmt_VERSION} in: ${fmt_DIR}")

add_subdirectory(thirdparty) # 处理子目录thirdparty，执行该目录下的CMakeLists.txt文件

# custom scoped cli11 target
add_library(basalt::cli11 INTERFACE IMPORTED)
set_property(TARGET basalt::cli11 PROPERTY INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/CLI11/include)

# custom scoped magic_enum target
add_library(basalt::magic_enum INTERFACE IMPORTED)
set_property(TARGET basalt::magic_enum PROPERTY INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/magic_enum/include)

# main library
add_library(basalt SHARED)

# List all header and source files with target_sources.
# This ensures, e.g., that QtCreator >= 6.0 properly recognize these headers to belong to the project.
# To support cmake < 3.13, use absolute paths (see: https://crascit.com/2016/01/31/enhanced-source-file-handling-with-target_sources/)

# target_sources(basalt PUBLIC ... PRIVATE ...)命令，用于为指定目标添加源文件。
# PUBLIC：表示这些文件是公共接口的一部分，其他依赖于该库的目标将能够访问这些头文件
# PRIVATE：表示这些文件是私有的，仅对该库的实现可见，其他依赖于该库的目标将不能访问这些源文件。
target_sources(basalt
  PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/calibration/aprilgrid.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/calibration/calibration_helper.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/calibration/cam_calib.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/calibration/cam_imu_calib.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/calibration/vignette.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/device/rs_t265.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/hash_bow/hash_bow.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/dataset_io.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/dataset_io_euroc.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/dataset_io_kitti.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/dataset_io_rosbag.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/dataset_io_uzh.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/marg_data_io.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/block_diagonal.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/imu_block.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/landmark_block.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/landmark_block_abs_dynamic.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/linearization_abs_qr.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/linearization_abs_sc.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/linearization_base.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/linearization_rel_sc.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/frame_to_frame_optical_flow.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/multiscale_frame_to_frame_optical_flow.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/optical_flow.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/patch.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/patch_optical_flow.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/patterns.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/accumulator.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/linearize.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/poses_linearize.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/poses_optimize.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/spline_linearize.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/spline_optimize.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/ba_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/cast_utils.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/common_types.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/filesystem.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/format.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/imu_types.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/keypoints.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/nfr.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/sim_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/system_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/test_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/time_utils.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/tracks.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/union_find.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/vio_config.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/vis_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/ba_base.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/landmark_database.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/marg_helper.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/nfr_mapper.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/sc_ba_base.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/sqrt_ba_base.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/sqrt_keypoint_vio.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/sqrt_keypoint_vo.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/vio_estimator.h
  PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src/calibration/aprilgrid.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/calibration/calibraiton_helper.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/calibration/vignette.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/io/dataset_io.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/io/marg_data_io.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/linearization/landmark_block.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/linearization/linearization_abs_qr.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/linearization/linearization_abs_sc.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/linearization/linearization_base.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/linearization/linearization_rel_sc.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/optical_flow/optical_flow.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/utils/keypoints.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/utils/system_utils.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/utils/time_utils.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/utils/vio_config.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/ba_base.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/landmark_database.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/marg_helper.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/nfr_mapper.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/sc_ba_base.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/sqrt_ba_base.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/sqrt_keypoint_vio.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/sqrt_keypoint_vo.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/vio_estimator.cpp
)

# PUBLIC：这些库不仅对basalt可见，而且对所有链接到basalt的目标也可见。这意味着如果其他目标链接了basalt，它们也会自动链接到这些库
# PRIVATE：这些库仅对basalt可见，对其他链接到basalt的目标不可见。
target_link_libraries(basalt
  PUBLIC ${STD_CXX_FS} basalt::opencv basalt::basalt-headers TBB::tbb
  PRIVATE basalt::magic_enum rosbag apriltag opengv nlohmann::json fmt::fmt)
# 为目标设置头文件包含路径和编译选项
target_include_directories(basalt PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_compile_definitions(basalt PUBLIC ${BASALT_COMPILE_DEFINITIONS})
#target_compile_definitions(basalt PUBLIC BASALT_DISABLE_ASSERTS)

add_executable(basalt_calibrate src/calibrate.cpp src/calibration/cam_calib.cpp)
target_link_libraries(basalt_calibrate basalt pangolin basalt::cli11)

add_executable(basalt_calibrate_imu src/calibrate_imu.cpp src/calibration/cam_imu_calib.cpp)
target_link_libraries(basalt_calibrate_imu basalt pangolin basalt::cli11)

add_executable(basalt_vio_sim src/vio_sim.cpp)
target_link_libraries(basalt_vio_sim basalt pangolin basalt::cli11)

add_executable(basalt_mapper_sim src/mapper_sim.cpp)
target_link_libraries(basalt_mapper_sim basalt pangolin basalt::cli11)

# mapper sim native doesn't use template free interface
if(BASALT_INSTANTIATIONS_DOUBLE)
  add_executable(basalt_mapper_sim_naive src/mapper_sim_naive.cpp)
  target_link_libraries(basalt_mapper_sim_naive basalt pangolin basalt::cli11)
endif()

add_executable(basalt_mapper src/mapper.cpp)
target_link_libraries(basalt_mapper basalt pangolin basalt::cli11)


add_executable(basalt_opt_flow src/opt_flow.cpp)
target_link_libraries(basalt_opt_flow basalt pangolin basalt::cli11)

add_executable(basalt_vio src/vio.cpp)
target_link_libraries(basalt_vio basalt pangolin basalt::cli11)

add_executable(basalt_time_alignment src/time_alignment.cpp)
target_link_libraries(basalt_time_alignment basalt pangolin basalt::cli11)

add_executable(basalt_kitti_eval src/kitti_eval.cpp)
target_link_libraries(basalt_kitti_eval basalt::basalt-headers basalt::cli11)

find_package(realsense2 QUIET)
if(realsense2_FOUND)
  add_executable(basalt_rs_t265_record src/rs_t265_record.cpp src/device/rs_t265.cpp)
  target_link_libraries(basalt_rs_t265_record basalt realsense2::realsense2 pangolin basalt::cli11)

  add_executable(basalt_rs_t265_vio src/rs_t265_vio.cpp src/device/rs_t265.cpp)
  target_link_libraries(basalt_rs_t265_vio basalt realsense2::realsense2 pangolin basalt::cli11)
endif()

# EXPORT BasaltTargets：将这些目标导出为BasaltTargets，可以在其他CMake项目中使用find_package或include引入这些目标
install(TARGETS basalt_calibrate basalt_calibrate_imu basalt_vio_sim basalt_mapper_sim basalt_mapper basalt_opt_flow basalt_vio basalt_kitti_eval basalt_time_alignment basalt
  EXPORT BasaltTargets
  RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
  LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
  ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)

if(BASALT_INSTANTIATIONS_DOUBLE)
  install(TARGETS basalt_mapper_sim_naive
    EXPORT BasaltTargets
    RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
    LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
    ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
endif()

if(realsense2_FOUND)
  install(TARGETS basalt_rs_t265_record basalt_rs_t265_vio
    EXPORT BasaltTargets
    RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
    LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
    ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
endif()

# 将当前CMake文件所在目录下的scripts目录中所有符合basalt_*.py 模式的文件路径存储在变量SCRIPTS_TO_INSTALL中
file(GLOB SCRIPTS_TO_INSTALL "${CMAKE_CURRENT_SOURCE_DIR}/scripts/basalt_*.py")
install(PROGRAMS ${SCRIPTS_TO_INSTALL} DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)

file(GLOB CONFIG_FILES "${CMAKE_CURRENT_SOURCE_DIR}/data/*.json")
install(FILES ${CONFIG_FILES}
  DESTINATION ${CMAKE_INSTALL_PREFIX}/etc/basalt)

# Replace install() to do-nothing macro.
macro(install)
endmacro()
# Include subproject (or any other CMake code) with "disabled" install().
enable_testing() # 启用测试功能
add_subdirectory(thirdparty/basalt-headers/test)
add_subdirectory(test)
# Restore original install() behavior.
macro(install)
  _install(${ARGN})
endmacro()
