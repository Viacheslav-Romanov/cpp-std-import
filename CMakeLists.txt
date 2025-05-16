cmake_minimum_required(VERSION 3.31)

# In the CMAKE's installation folder (e.g. /opt/homebrew/Cellar/cmake/4.0.2)
# replace an entry "-print-file-name=libc++.modules.json"
# with "-print-file-name=../../c++/libc++.modules.json"
# in Clang-CXX-CXXImportStd.cmake
# an issue is opened on llvm side https://github.com/llvm/llvm-project/issues/120215

# After googling many moons for the right ID I stumbled upon a discussion https://discourse.cmake.org/t/import-std-appears-to-be-broken-in-release-4-0-2/14062
# find it for your CMAKE version at https://github.com/Kitware/CMake/blob/v4.0.2/Help/dev/experimental.rst
set(CMAKE_EXPERIMENTAL_CXX_IMPORT_STD "a9e1cf81-9932-4810-974b-6eccaf14e457")

project(cpp-std-import LANGUAGES CXX)

set(CMAKE_CXX_IMPORT_STD ON)
set(CMAKE_CXX_IMPLICIT_MODULES ON)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_MODULE_STD ON)
set(CMAKE_CXX_SCAN_FOR_MODULES ON)
set(CMAKE_CXX_STANDARD 26)
set(CMAKE_CXX_EXTENSIONS ON)  # let experimental feature work

add_executable(cpp-std-import main.cpp)

target_sources(cpp-std-import PRIVATE
        FILE_SET CXX_MODULES
        FILES module1.cppm
)

# Add linking against the C++ standard library
if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        target_compile_options(cpp-std-import PRIVATE
                -fmodules
                -fcxx-modules
                -fbuiltin-module-map
                -fimplicit-modules
                -fimplicit-module-maps
        )
        target_link_options(cpp-std-import PRIVATE
                -fmodules
                -fcxx-modules
                -fbuiltin-module-map
                -fimplicit-modules
                -fimplicit-module-maps
        )
        # For Clang on macOS, we typically link against libc++
        target_link_libraries(cpp-std-import PRIVATE c++)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    # probably similar dances you need to do on your linux machine idk
    # For GCC, link against libstdc++
    target_link_libraries(cpp-std-import PRIVATE stdc++)
endif()

target_compile_features(cpp-std-import PUBLIC cxx_std_26) # somehow important

# To build all this goodness need to follow the following steps:
# rm -rf cmake-build-*
# cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_MAKE_PROGRAM=ninja -DCMAKE_CXX_COMPILER=/opt/homebrew/Cellar/llvm/20.1.4_1/bin/clang-20 -G Ninja -S . -B cmake-build-release
# cmake --build cmake-build-release
