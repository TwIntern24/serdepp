cmake_minimum_required(VERSION 3.20)
project(serdepp VERSION 0.1.4 LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(CTest)
enable_testing()

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        add_compile_options(-fprofile-instr-generate -fcoverage-mapping --coverage)
    endif()
endif()

include(FetchContent)
set(FETCHCONTENT_BASE_DIR ${CMAKE_SOURCE_DIR}/.cache)
set(FETCHCONTENT_FULLY_DISCONNECTED ON)
FetchContent_Declare(
    fmt
    GIT_REPOSITORY https://gitee.com/kj-smart/fmt.git
    GIT_TAG 12.1.0
    GIT_SHALLOW TRUE
    GIT_PROGRESS TRUE
)
FetchContent_MakeAvailable(fmt)
FetchContent_Declare(
    magic_enum
    GIT_REPOSITORY https://gitee.com/kj-smart/magic_enum.git
    GIT_TAG  v0.9.7
    GIT_SHALLOW TRUE
    GIT_PROGRESS TRUE
)
FetchContent_MakeAvailable(magic_enum)

FetchContent_Declare(
    nameof
    GIT_REPOSITORY https://gitee.com/kj-smart/nameof.git
    GIT_TAG  v0.10.4
    GIT_SHALLOW TRUE
    GIT_PROGRESS TRUE
)
FetchContent_MakeAvailable(nameof)


FetchContent_Declare(
    nlohmann_json
    GIT_REPOSITORY https://gitee.com/kj-smart/json.git
    GIT_TAG  v3.12.0
    GIT_SHALLOW TRUE
    GIT_PROGRESS TRUE
)
FetchContent_MakeAvailable(nlohmann_json)

FetchContent_Declare(
    yaml_cpp
    GIT_REPOSITORY https://gitee.com/kj-smart/yaml-cpp.git
    GIT_TAG  0.8.0
    GIT_SHALLOW TRUE
    GIT_PROGRESS TRUE
)
FetchContent_MakeAvailable(yaml_cpp)


set(EXPORT_NAMESPACE "${PROJECT_NAME}::")
add_library(${PROJECT_NAME} INTERFACE)
add_library(${EXPORT_NAMESPACE}${PROJECT_NAME} ALIAS ${PROJECT_NAME})
target_include_directories(
  ${PROJECT_NAME} INTERFACE $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)


set(serdepp_global_deps
    nlohmann_json  yaml-cpp magic_enum nameof fmt
)
target_link_libraries(${PROJECT_NAME} INTERFACE ${serdepp_global_deps})

option(SERDEPP_BUILD_EXAMPLES "Build examples" ON)
if(SERDEPP_BUILD_EXAMPLES)
        add_executable(serde_example1 examples/example1.cpp)
    target_link_libraries(serde_example1 PRIVATE ${PROJECT_NAME})

            
        
        
        
    add_executable(parse_file_example examples/parse_file.cpp)
    target_compile_definitions(parse_file_example PRIVATE yaml_file="${PROJECT_SOURCE_DIR}/examples/test.yaml")
    target_link_libraries(parse_file_example PRIVATE ${PROJECT_NAME})

        
        
        
        
        endif()

option(SERDEPP_BUILD_TESTING "Build tests" OFF)
if(SERDEPP_BUILD_TESTING AND BUILD_TESTING)
    add_executable(unittest
        tests/serializer.cpp
        tests/rapid_json.cpp
        tests/yaml_cpp.cpp
        tests/toml11.cpp
        tests/reflection.cpp
        tests/nlohmann_json.cpp
    )

    target_link_libraries(unittest
        PRIVATE serdepp ${serdepp_global_deps}
    )

        add_test(NAME unittest COMMAND unittest)
endif()

option(SERDEPP_BUILD_BENCHMARKS "Build benchmarks" OFF)
if(SERDEPP_BUILD_BENCHMARKS)
    add_executable(benchmark benchmark/benchmark.cpp)
    target_link_libraries(benchmark PRIVATE serdepp benchmark::benchmark ${serdepp_global_deps})

    add_executable(data_type_benchmark benchmark/data_type_benchmark.cpp)
    target_link_libraries(data_type_benchmark PRIVATE serdepp benchmark::benchmark ${serdepp_global_deps})

    add_executable(syntax_sugar_benchmark benchmark/syntax_benchmark.cpp)
    target_link_libraries(syntax_sugar_benchmark PRIVATE serdepp benchmark::benchmark ${serdepp_global_deps})
endif()

include(GNUInstallDirs)
