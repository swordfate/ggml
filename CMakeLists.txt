# 设置CMake最低版本要求为3.14，因为需要使用add_link_options和隐式目标目录等特性
cmake_minimum_required(VERSION 3.14)

# 定义项目名为"ggml"，支持C和C++语言
project("ggml" C CXX)

# 包含其他CMake模块
include(CheckIncludeFileCXX)

# 生成compile_commands.json文件，这对代码提示和静态分析工具很有用
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# 如果不是Xcode或MSVC环境，且没有设置构建类型，则默认设置为Release构建
if (NOT XCODE AND NOT MSVC AND NOT CMAKE_BUILD_TYPE)
    # 强制设置构建类型为Release
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
    # 设置可选的构建类型
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# 检查是否是独立构建（即ggml是主项目）
if (CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    # 设置为独立构建模式
    set(GGML_STANDALONE ON)
    # 设置可执行文件的输出目录
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
else()
    # 设置为非独立构建模式（作为子项目）
    set(GGML_STANDALONE OFF)
endif()

# WebAssembly编译相关设置
if (EMSCRIPTEN)
    # 在WebAssembly环境下默认不构建共享库
    set(BUILD_SHARED_LIBS_DEFAULT OFF)
    # 设置是否将WASM嵌入到生成的ggml.js中的选项
    option(GGML_WASM_SINGLE_FILE "ggml: embed WASM inside the generated ggml.js" ON)
else()
    # 非WebAssembly环境下的共享库设置
    if (MINGW)
        # MinGW环境下默认不构建共享库
        set(BUILD_SHARED_LIBS_DEFAULT OFF)
    else()
        # 其他环境默认构建共享库
        set(BUILD_SHARED_LIBS_DEFAULT ON)
    endif()
endif()

# 是否构建共享库的选项
option(BUILD_SHARED_LIBS "ggml: build shared libraries" ${BUILD_SHARED_LIBS_DEFAULT})

#
# option list
#

# TODO: mark all options as advanced when not GGML_STANDALONE

if (APPLE)
    set(GGML_METAL_DEFAULT ON)
    set(GGML_BLAS_DEFAULT ON)
    set(GGML_BLAS_VENDOR_DEFAULT "Apple")
else()
    set(GGML_METAL_DEFAULT OFF)
    set(GGML_BLAS_DEFAULT OFF)
    set(GGML_BLAS_VENDOR_DEFAULT "Generic")
endif()

if (CMAKE_CROSSCOMPILING)
    set(GGML_NATIVE_DEFAULT OFF)
else()
    set(GGML_NATIVE_DEFAULT ON)
endif()

# general
option(GGML_STATIC "ggml: static link libraries"         OFF)
option(GGML_NATIVE "ggml: enable -march=native flag"     ${GGML_NATIVE_DEFAULT})
option(GGML_LTO    "ggml: enable link time optimization" OFF)
option(GGML_CCACHE "ggml: use ccache if available"       ON)

# debug
option(GGML_ALL_WARNINGS           "ggml: enable all compiler warnings"                   ON)
option(GGML_ALL_WARNINGS_3RD_PARTY "ggml: enable all compiler warnings in 3rd party libs" OFF)
option(GGML_GPROF                  "ggml: enable gprof"                                   OFF)

# build
option(GGML_FATAL_WARNINGS    "ggml: enable -Werror flag"    OFF)

# sanitizers
option(GGML_SANITIZE_THREAD    "ggml: enable thread sanitizer"    OFF)
option(GGML_SANITIZE_ADDRESS   "ggml: enable address sanitizer"   OFF)
option(GGML_SANITIZE_UNDEFINED "ggml: enable undefined sanitizer" OFF)

# instruction set specific
if (GGML_NATIVE OR NOT GGML_NATIVE_DEFAULT)
    set(INS_ENB OFF)
else()
    set(INS_ENB ON)
endif()

option(GGML_CPU_HBM     "ggml: use memkind for CPU HBM" OFF)

option(GGML_AVX         "ggml: enable AVX"              ${INS_ENB})
option(GGML_AVX2        "ggml: enable AVX2"             ${INS_ENB})
option(GGML_AVX512      "ggml: enable AVX512"           OFF)
option(GGML_AVX512_VBMI "ggml: enable AVX512-VBMI"      OFF)
option(GGML_AVX512_VNNI "ggml: enable AVX512-VNNI"      OFF)
option(GGML_AVX512_BF16 "ggml: enable AVX512-BF16"      OFF)
option(GGML_FMA         "ggml: enable FMA"              ${INS_ENB})
if (NOT MSVC)
    option(GGML_F16C    "ggml: enable F16C"             ${INS_ENB}) # in MSVC F16C is implied with AVX2/AVX512
endif()
option(GGML_LASX        "ggml: enable lasx"             ON)
option(GGML_LSX         "ggml: enable lsx"              ON)
option(GGML_SVE         "ggml: enable SVE"              OFF)

if (WIN32)
    set(GGML_WIN_VER "0x602" CACHE STRING "ggml: Windows Version")
endif()

# ggml core
set(GGML_SCHED_MAX_COPIES  "4" CACHE STRING "ggml: max input copies for pipeline parallelism")

# 3rd party libs / backends
option(GGML_ACCELERATE                      "ggml: enable Accelerate framework"               ON)
option(GGML_BLAS                            "ggml: use BLAS"                                  ${GGML_BLAS_DEFAULT})
set(GGML_BLAS_VENDOR ${GGML_BLAS_VENDOR_DEFAULT} CACHE STRING
                                            "ggml: BLAS library vendor")
option(GGML_LLAMAFILE                       "ggml: use LLAMAFILE"                             OFF)

option(GGML_CUDA                            "ggml: use CUDA"                                  OFF)
option(GGML_MUSA                            "ggml: use MUSA"                                  OFF)
option(GGML_CUDA_FORCE_DMMV                 "ggml: use dmmv instead of mmvq CUDA kernels"     OFF)
option(GGML_CUDA_FORCE_MMQ                  "ggml: use mmq kernels instead of cuBLAS"         OFF)
option(GGML_CUDA_FORCE_CUBLAS               "ggml: always use cuBLAS instead of mmq kernels"  OFF)
set   (GGML_CUDA_DMMV_X   "32" CACHE STRING "ggml: x stride for dmmv CUDA kernels")
set   (GGML_CUDA_MMV_Y     "1" CACHE STRING "ggml: y block size for mmv CUDA kernels")
option(GGML_CUDA_F16                        "ggml: use 16 bit floats for some calculations"   OFF)
set   (GGML_CUDA_KQUANTS_ITER "2" CACHE STRING
                                            "ggml: iters./thread per block for Q2_K/Q6_K")
set   (GGML_CUDA_PEER_MAX_BATCH_SIZE "128" CACHE STRING
                                            "ggml: max. batch size for using peer access")
option(GGML_CUDA_NO_PEER_COPY               "ggml: do not use peer to peer copies"            OFF)
option(GGML_CUDA_NO_VMM                     "ggml: do not try to use CUDA VMM"                OFF)
option(GGML_CUDA_FA_ALL_QUANTS              "ggml: compile all quants for FlashAttention"     OFF)
option(GGML_CUDA_USE_GRAPHS                 "ggml: use CUDA graphs (llama.cpp only)"          OFF)

option(GGML_HIPBLAS                         "ggml: use hipBLAS"                               OFF)  # AMD HIP支持
option(GGML_HIP_UMA                         "ggml: use HIP unified memory architecture"       OFF)  # HIP统一内存架构
option(GGML_VULKAN                          "ggml: use Vulkan"                                OFF)  # Vulkan支持
option(GGML_VULKAN_CHECK_RESULTS            "ggml: run Vulkan op checks"                      OFF)  # Vulkan操作检查
option(GGML_VULKAN_DEBUG                    "ggml: enable Vulkan debug output"                OFF)  # Vulkan调试输出
option(GGML_VULKAN_MEMORY_DEBUG             "ggml: enable Vulkan memory debug output"         OFF)  # Vulkan内存调试
option(GGML_VULKAN_SHADER_DEBUG_INFO        "ggml: enable Vulkan shader debug info"           OFF)  # Vulkan着色器调试信息
option(GGML_VULKAN_PERF                     "ggml: enable Vulkan perf output"                 OFF)  # Vulkan性能输出
option(GGML_VULKAN_VALIDATE                 "ggml: enable Vulkan validation"                  OFF)  # Vulkan验证
option(GGML_VULKAN_RUN_TESTS                "ggml: run Vulkan tests"                          OFF)  # 运行Vulkan测试
option(GGML_KOMPUTE                         "ggml: use Kompute"                               OFF)  # Kompute支持
option(GGML_METAL                           "ggml: use Metal"                                 ${GGML_METAL_DEFAULT})  # Apple Metal支持
option(GGML_METAL_NDEBUG                    "ggml: disable Metal debugging"                   OFF)  # 禁用Metal调试
option(GGML_METAL_SHADER_DEBUG              "ggml: compile Metal with -fno-fast-math"         OFF)  # Metal着色器调试
option(GGML_METAL_EMBED_LIBRARY             "ggml: embed Metal library"                       ${GGML_METAL})  # 嵌入Metal库
set   (GGML_METAL_MACOSX_VERSION_MIN "" CACHE STRING "ggml: metal minimum macOS version")  # Metal最低macOS版本
set   (GGML_METAL_STD "" CACHE STRING "ggml: metal standard version (-std flag)")  # Metal标准版本
option(GGML_OPENMP                          "ggml: use OpenMP"                                ON)   # OpenMP支持
option(GGML_RPC                             "ggml: use RPC"                                   OFF)  # RPC支持
option(GGML_SYCL                            "ggml: use SYCL"                                  OFF)  # SYCL支持
option(GGML_SYCL_F16                        "ggml: use 16 bit floats for sycl calculations"   OFF)  # SYCL半精度浮点
set   (GGML_SYCL_TARGET "INTEL" CACHE STRING "ggml: sycl target device")  # SYCL目标设备

# 额外构建产物选项
option(GGML_BUILD_TESTS    "ggml: build tests"    ${GGML_STANDALONE})  # 是否构建测试程序
option(GGML_BUILD_EXAMPLES "ggml: build examples" ${GGML_STANDALONE})  # 是否构建示例程序

#
# dependencies
#

# 设置基本编译选项
set(CMAKE_C_STANDARD 11)                  # 使用C11标准
set(CMAKE_C_STANDARD_REQUIRED true)       # 强制要求C11标准

# 根据是否使用SYCL来设置C++标准
if (GGML_SYCL)
    set(CMAKE_CXX_STANDARD 17)           # SYCL需要C++17
else()
    set(CMAKE_CXX_STANDARD 11)           # 否则使用C++11
endif()
set(CMAKE_CXX_STANDARD_REQUIRED true)    # 强制要求所选的C++标准

# 线程相关设置
set(THREADS_PREFER_PTHREAD_FLAG ON)      # 优先使用pthread标志
find_package(Threads REQUIRED)           # 查找线程库
link_libraries(Threads::Threads)         # 链接线程库

#
# build the library
#

add_subdirectory(src)  # 添加src子目录，构建ggml核心库

#
# tests and examples
#

# 根据选项决定是否构建测试和示例
if (GGML_BUILD_TESTS)
    enable_testing()                     # 启用测试
    add_subdirectory(tests)             # 添加测试目录
endif()

if (GGML_BUILD_EXAMPLES)
    add_subdirectory(examples)          # 添加示例目录
endif()

#
# install
#

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# 所有公共头文件列表
set(GGML_PUBLIC_HEADERS
    include/ggml.h
    include/ggml-alloc.h
    include/ggml-backend.h
    include/ggml-blas.h
    include/ggml-cann.h
    include/ggml-cuda.h
    include/ggml.h
    include/ggml-kompute.h
    include/ggml-metal.h
    include/ggml-rpc.h
    include/ggml-sycl.h
    include/ggml-vulkan.h)

set_target_properties(ggml PROPERTIES PUBLIC_HEADER "${GGML_PUBLIC_HEADERS}")  # 设置ggml目标的公共头文件属性
#if (GGML_METAL)
#    set_target_properties(ggml PROPERTIES RESOURCE "${CMAKE_CURRENT_SOURCE_DIR}/src/ggml-metal.metal")
#endif()
install(TARGETS ggml PUBLIC_HEADER)  # 安装ggml库及其公共头文件

if (BUILD_SHARED_LIBS)
    install(TARGETS ggml LIBRARY)  # 如果构建共享库，安装共享库文件
endif()

if (GGML_METAL)
    install(
        FILES src/ggml-metal.metal  # 安装Metal源文件
        PERMISSIONS
            OWNER_READ
            OWNER_WRITE
            GROUP_READ
            WORLD_READ
        DESTINATION ${CMAKE_INSTALL_BINDIR})

    if (NOT GGML_METAL_EMBED_LIBRARY)
        install(
            FILES ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/default.metallib  # 安装编译好的Metal库
            DESTINATION ${CMAKE_INSTALL_BINDIR}
        )
    endif()
endif()

if (GGML_STANDALONE)
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/ggml.pc.in  # 配置pkg-config文件模板
        ${CMAKE_CURRENT_BINARY_DIR}/ggml.pc
        @ONLY)

    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/ggml.pc  # 安装pkg-config文件
        DESTINATION share/pkgconfig)
endif()
