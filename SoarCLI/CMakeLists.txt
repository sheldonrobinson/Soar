cmake_minimum_required(VERSION 3.22)
project(soar_cli
    VERSION 9.6.3
    LANGUAGES CXX)

add_executable(soarcli soar_cli.cpp)
target_link_libraries(soarcli soar::soar)

install(TARGETS soarcli
    DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)
