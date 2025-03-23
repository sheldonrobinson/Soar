cmake_minimum_required(VERSION 3.22)
project(performance_tests
    LANGUAGES CXX)

# The executable is also called PerformanceTests by scons.
set(EXEC_NAME PerformanceTests)
add_executable(${EXEC_NAME} PerformanceTests.cpp)
target_link_libraries(${EXEC_NAME} soar)
target_compile_features(${EXEC_NAME} PUBLIC
    cxx_std_17
)

file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/TestAgents/ DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/do_performance_test.sh DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

enable_testing()

if(BUILD_TESTING)
    add_test(NAME test_performance COMMAND do_performance_test.sh)
endif()
