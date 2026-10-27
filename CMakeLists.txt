cmake_minimum_required(VERSION 4.3)
project(webtoon
        VERSION 0
        COMPAT_VERSION 0
        SPDX_LICENSE Apache-2.0
        DESCRIPTION "C library and command line reference implementation of a file format, .wef"
        HOMEPAGE_URL "https://github.com/TheMonHub/webtoon"
        LANGUAGES C
)

set(CMAKE_C_STANDARD 23)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

set(CMAKE_C_CLANG_TIDY
        clang-tidy;
        -format-style='file';
        -header-filter=.;
        -checks=bugprone-*,performance-*,clang-analyzer-*,cert-*;
        -warnings-as-errors=*;
)

# OPTIONS

option(WEBTOON_BUILD_CLI "Build a command line executable.")

# LIBRARY

find_package(ImageMagick REQUIRED COMPONENTS MagickWand)

add_library(webtoon STATIC
        lib/src/library.c
        lib/include/webtoon/library.h
)

target_link_libraries(webtoon
        PRIVATE
        ImageMagick::MagickWand
)

target_include_directories(webtoon PUBLIC
        lib/include
)

# APP

if (NOT WEBTOON_BUILD_CLI)
    message(STATUS "Skipping CLI build.")
    RETURN()
endif ()

add_executable(webtoon_cli
        cli/src/main.c
        cli/src/main.h
)

target_link_libraries(webtoon_cli PRIVATE
        webtoon
)