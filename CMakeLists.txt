cmake_minimum_required(VERSION 3.16)

if (NOT DEFINED FUN_TIMES_BOOST_VERSION)
    set(FUN_TIMES_BOOST_VERSION "1.74.0" CACHE STRING "Boost Version")
endif ()

include(FetchContent)
include(cmake/RequireBoost.cmake)
include(cmake/BuildCMakeContent.cmake)

project(beast_fun_times)

option(ENABLE_TESTING "" ON)

if (NOT DEFINED CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 17)
endif ()

list(APPEND CMAKE_PREFIX_PATH ${CMAKE_CURRENT_BINARY_DIR}/dependencies/install)
if (NOT DEFINED BOOST_ROOT)
    #    RequireBoost(PREFIX ${CMAKE_CURRENT_BINARY_DIR}/dependencies/install VERSION 1.73.0 COMPONENTS system)
    set(boost_components system thread)
    if (FUN_TIMES_BOOST_VERSION VERSION_GREATER "1.71.0")
        list(APPEND boost_components container)
    endif()
    RequireBoost(PREFIX "${CMAKE_CURRENT_BINARY_DIR}/dependencies/install/boost-${FUN_TIMES_BOOST_VERSION}" VERSION "${FUN_TIMES_BOOST_VERSION}"
            COMPONENTS
            ${boost_components})
else()
    message(STATUS "[dependencies] BOOST_ROOT=${BOOST_ROOT}")
endif ()

if (ENABLE_TESTING)
    FetchContent_Declare(catch2
            GIT_REPOSITORY https://github.com/catchorg/Catch2.git
            GIT_TAG v2.12.1)
    BuildCmakeContent(catch2 Catch2 CMAKE_ARGS
            "-DCATCH_BUILD_TESTING=OFF"
            "-DCATCH_INSTALL_DOCS=OFF"
            "-DCATCH_INSTALL_HELPERS=OFF")
    find_package(Catch2 CONFIG)
endif ()

FetchContent_Declare(fmtlib
        GIT_REPOSITORY git@github.com:fmtlib/fmt.git
        GIT_TAG 7.0.3)
BuildCmakeContent(fmtlib fmt
    CMAKE_ARGS
        "-DFMT_TEST=OFF"
        "-DFMT_DOC=OFF")

FetchContent_Declare(spdlog
        GIT_REPOSITORY  https://github.com/gabime/spdlog.git
        GIT_TAG v1.8.1)
BuildCMakeContent(spdlog spdlog
        CMAKE_ARGS
        -DSPDLOG_BUILD_EXAMPLE=OFF
        -DSPDLOG_BUILD_TESTS=OFF
        -DSPDLOG_INSTALL=ON
        -DSPDLOG_FMT_EXTERNAL=ON)


FetchContent_Declare(nlohmann_jsonlib
        GIT_REPOSITORY git@github.com:nlohmann/json.git
        GIT_TAG v3.9.1)
BuildCmakeContent(nlohmann_jsonlib nlohmann_json
    CMAKE_ARGS
        "-DJSON_BuildTests=OFF"
        "-DJSON_Install=ON")

if (FUN_TIMES_BOOST_VERSION VERSION_GREATER "1.71.0")

FetchContent_Declare(boost_json_lib
        GIT_REPOSITORY git@github.com:CPPAlliance/json.git
        GIT_TAG 0da206074fbee99877ae82bb73afbc567e19a7b6)
BuildCmakeContent(boost_json_lib boost_json
        CMAKE_ARGS
        "-DBOOST_JSON_STANDALONE=OFF"
        "-DBOOST_JSON_BUILD_TESTS=OFF"
        "-DBOOST_JSON_BUILD_FUZZERS=OFF"
        "-DBOOST_JSON_BUILD_EXAMPLES=OFF"
        "-DBOOST_JSON_BUILD_BENCHMARKS=OFF")

add_compile_definitions(
        "BOOST_ASIO_NO_DEPRECATED=1"
        "BOOST_ASIO_DISABLE_CONCEPTS=1")
endif()

set(Boost_USE_STATIC_LIBS ON)
find_package(Boost CONFIG REQUIRED COMPONENTS ${boost_components})
find_package(Threads)
find_package(fmt CONFIG)
find_package(spdlog CONFIG)
find_package(nlohmann_json CONFIG)
find_package(OpenSSL)
if (FUN_TIMES_BOOST_VERSION VERSION_GREATER "1.71.0")
find_package(boost_json)
endif()
add_subdirectory(lib)

add_subdirectory(pre-cxx20)
if (CMAKE_CXX_STANDARD GREATER_EQUAL 20)
    add_subdirectory(cxx20)
endif ()

