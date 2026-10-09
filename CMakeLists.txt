cmake_minimum_required(VERSION 4.0)

project (CoreKV
        VERSION 0.1.0
        LANGUAGES CXX
)

# C++ standard.
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Executable
add_executable(core_kv
        src/main.cpp
)

# Compiler warnings.
target_compile_options(core_kv PRIVATE
        -Wall
        -Wextra
        -Wpedantic
)
