# -DCMAKE_BUILD_TYPE=Debug
# -DCMAKE_BUILD_TYPE=Release
# -DCMAKE_VERBOSE_MAKEFILE=ON
# -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++
#
# ASan:
#  https://software.intel.com/en-us/articles/address-sanitizer-asan-tool-to-debug-memory-corruptions-in-chrome-os
#  http://www.youtube.com/watch?v=V2_80g0eOMc
#  http://www.youtube.com/watch?v=capbD_aRz40


project(asan-check)
cmake_minimum_required(VERSION 2.8)

# conf
set (LCB_USE_ASAN Y)

set(Extern "../../../extern/")
set(GMock "../../../extern/gmock-1.6.0/fused-src")
set(Source "source/")
set(testsRoot "./tests")
set(BigExtern "/opt/big-3rdparty/")

# Test framework
include_directories(${GMock})
include_directories(${Source})
include_directories(${Extern}/loki-0.1.7/include)
include_directories(${Extern})
include_directories(${BigExtern}/boost_1_57_0)

# ASan,TSan and MSan
IF(LCB_USE_ASAN)
    # GCC: troubles with symbols
    #ASAN_SYMBOLIZER_PATH=$(which llvm-symbolizer-3.4)
    #/usr/bin/llvm-symbolizer-3.4

    # TOOLS:
    #
    # https://code.google.com/p/address-sanitizer/wiki/HowToBuild
    # http://cameo54321.blogspot.ru/2014/02/installing-cmake-288-or-higher-on.html
    # clang 3.1 >= need for ASan Ub12.04 -> 3.0
    #
    # !!https://solarianprogrammer.com/2013/01/17/building-clang-libcpp-ubuntu-linux/
    # Try it! https://github.com/pacs-course/pacs/wiki/Instructions-to-install-clang-3.1-on-ubuntu-12.04.1-and-12.10
    # http://stackoverflow.com/questions/17657261/how-to-install-clang-pre-built-binaries-ubuntu-12-04
    # http://sohanikh.blogspot.ru/2013/02/how-to-build-llvm-from-source-on-ubuntu.html
    #add_definitions(  -Wall -g -fsanitize=address)
    # https://developer.mozilla.org/en-US/docs/Mozilla/Testing/Firefox_and_Address_Sanitizer

    # Static analyser http://clang-analyzer.llvm.org/
    # https://github.com/couchbase/couchnode/blob/master/deps/lcb/cmake/Modules/GetLibcouchbaseFlags.cmake
    # http://blogs.perl.org/users/rurban/2011/11/adventures-with-clang-and-asan.html
    # main: address - thread, memory and ub not work in i686, only x86_64
    #-fno-omit-frame-pointer -O1 -g
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -fno-omit-frame-pointer -fsanitize=address")
    #-fno-omit-frame-pointer -O1
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -fno-omit-frame-pointer -fsanitize=address")
    # -undefined dynamic_lookup
    SET(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS}  -fsanitize=address")
    SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=address")
ENDIF()

# link the libraries to the executable
if (UNIX)
    #add_definitions(  -Wall) # not working
    # http://stackoverflow.com/questions/15610404/gcc-does-not-give-any-warning-when-converting-from-64-bit-to-32-bit
    #-arch x86_64  # Apple's gcc
    # -Wshorten-64-to-32
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}  -Wall")

    # gcc 4.8 ubuntu 12 http://llvm.org/docs/GettingStarted.html#getting-a-modern-host-c-toolchain
    # cmake .. -DCMAKE_CXX_LINK_FLAGS="-Wl,-rpath,$HOME/toolchains/lib64 -L$HOME/toolchains/lib64"
    # http://llvm.org/docs/GettingStarted.html#getting-a-modern-host-c-toolchain

    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        link_directories(${BigExtern}/boost_1_57_0/stage_gcc/lib)
        link_libraries(boost_system boost_thread pthread rt)  # !! pthread and rt last!!
    else()
        link_directories(${BigExtern}/boost_1_57_0/stage_x64/lib)
        link_libraries(boost_system boost_thread pthread rt)  # !! pthread and rt last!!
    endif()
endif()

message("CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")

# Sources
file( GLOB set0 ${Source}/heart/*.h)
file( GLOB set2
    ${testsRoot}/*.cc
    ${testsRoot}/bug_report/*.cc)

add_executable(unittest.bin
    ${Source}/mains/launcher_all.cc
    ${GMock}/gmock-gtest-all.cc
    ${set0}
    ${set2}
)

