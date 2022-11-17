cmake_minimum_required(VERSION 3.9)

project(uap-cpp)

include(FetchContent)

FetchContent_Declare(
        uap-core
        URL https://github.com/ua-parser/uap-core/archive/refs/tags/v0.6.9.tar.gz
)
FetchContent_MakeAvailable(uap-core)

FetchContent_Declare(
        yaml-cpp
        GIT_REPOSITORY https://github.com/jbeder/yaml-cpp.git
        GIT_TAG yaml-cpp-0.7.0
)
FetchContent_MakeAvailable(yaml-cpp)

FetchContent_Declare(
        re2
        GIT_REPOSITORY https://github.com/google/re2.git
        GIT_TAG 2022-06-01
        GIT_SHALLOW TRUE
)
FetchContent_MakeAvailable(re2)

FetchContent_Declare(
        GTest
        GIT_REPOSITORY https://github.com/google/googletest
        GIT_TAG e2239ee6043f73722e7aa812a459f54a28552929 # v1.11.0
)

FetchContent_MakeAvailable(GTest)

IF(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
    SET(CMAKE_BUILD_TYPE Release)
ENDIF(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -Wall -Werror -fPIC")

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0")



set(LIB_SOURCES
        UaParser.cpp
        internal/AlternativeExpander.cpp
        internal/Pattern.cpp
        internal/ReplaceTemplate.cpp
        internal/SnippetIndex.cpp)

#find_package(yaml-cpp REQUIRED)
#message(${YAML_CPP_INCLUDE_DIR})
#include_directories(${YAML_CPP_INCLUDE_DIR})

#find_package(re2)

# this is the "object library" target: compiles the sources only once
add_library(objlib OBJECT ${LIB_SOURCES})
target_link_libraries(objlib PRIVATE yaml-cpp re2)

# shared libraries need PIC
set_property(TARGET objlib PROPERTY POSITION_INDEPENDENT_CODE 1)

# shared and static libraries built from the same object files
add_library(uap-cpp-static STATIC $<TARGET_OBJECTS:objlib>)
set_target_properties(uap-cpp-static PROPERTIES OUTPUT_NAME uaparser_cpp)
target_link_libraries(uap-cpp-static PUBLIC yaml-cpp re2)
target_compile_definitions(uap-cpp-static PUBLIC
        UA_CORE_ROOT="${uap-core_SOURCE_DIR}"
)

#add_library(uap-cpp-shared SHARED $<TARGET_OBJECTS:objlib>)
#set_target_properties(uap-cpp-shared PROPERTIES OUTPUT_NAME uaparser_cpp)

#add_library(uap-cpp-core re2 yaml-cpp )

set(TEST_SOURCES
        UaParserTest.cpp)

#find_package(GTest)

add_executable(tests ${TEST_SOURCES} #[[$<TARGET_OBJECTS:objlib>]])
set_target_properties(tests PROPERTIES OUTPUT_NAME UaParserTest)

target_link_libraries(tests gtest pthread uap-cpp-static)


#set(BENCH_SOURCES
#        benchmarks/UaParserBench.cpp)
#
#add_executable(bench ${BENCH_SOURCES} $<TARGET_OBJECTS:objlib>)
#set_target_properties(bench PROPERTIES OUTPUT_NAME UaParserBench)
#
#target_link_libraries(bench re2 yaml-cpp pthread)
