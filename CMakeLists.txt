cmake_minimum_required(VERSION 3.22)
project(soar
  VERSION 9.6.3
  LANGUAGES CXX C)

# find_package(SQLite3 REQUIRED)

add_library(SQLite3_objs OBJECT SQLite3/sqlite3.c)

target_compile_definitions(SQLite3_objs INTERFACE  
								   SQLITE_DQS=0
								   SQLITE_MAX_EXPR_DEPTH=0
								   SQLITE_DEFAULT_MEMSTATUS=0
								   SQLITE_DEFAULT_SYNCHRONOUS=1
								   SQLITE_DEFAULT_WAL_SYNCHRONOUS=1
								   SQLITE_THREADSAFE=1
								   SQLITE_DEFAULT_AUTOVACUUM=2
								   SQLITE_OMIT_LOAD_EXTENSION=1
								   SQLITE_ENABLE_MATH_FUNCTIONS=1
								   SQLITE_TEMP_STORE=2
								   SQLITE_ENABLE_FTS5=1
								   SQLITE_ENABLE_MEMORY_MANAGEMENT=1)
								   

set_property(TARGET SQLite3_objs
             PROPERTY POSITION_INDEPENDENT_CODE True
			 PROPERTY C_STANDARD 11
			 PROPERTY LINKER_LANGUAGE C)


# All options should be turned off by default to avoid building unnecessary
# components in downstream projects relying on cmake's FetchContent module.
# FetchContent does not allow to set options for dependencies.
option(BUILD_TESTS "Build tests" OFF)
option(CLI "Build Soar command line interface" OFF)

# ASAN only works in debug mode.
option(ASAN "Enable AddressSanitizer" OFF)
option(SVS "SVS Enabled" OFF)

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  message(STATUS "Soar: Insallation prefix not set. Defaulting to ${CMAKE_INSTALL_PREFIX}")
  set_property(CACHE CMAKE_INSTALL_PREFIX PROPERTY VALUE "${CMAKE_CURRENT_SOURCE_DIR}/install")
endif()

option(SYSTEM_INSTALL "Install to system directories" OFF)

if(SYSTEM_INSTALL)
    include(GNUInstallDirs)
else()
    set(CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_SOURCE_DIR}/install" CACHE PATH "Installation Directory" FORCE)
endif()

# Add your source files
set(SOURCES
  Core/ClientSML/ClientSML.cxx
  Core/ConnectionSML/ConnectionSML.cxx
  Core/ElementXML/ElementXML.cxx
  Core/KernelSML/KernelSML.cxx
  Core/SoarKernel/SoarKernel.cxx
  Core/shared/shared.cxx
  Core/CLI/CommandLineInterface.cxx
)

file(GLOB_RECURSE HEADERS
  "Core/CLI/src/*.h"
  "Core/ClientSML/src/*.h"
  "Core/ConnectionSML/src/*.h"
  "Core/ElementXML/src/*.h"
  "Core/KernelSML/src/*.h"
  "Core/SoarKernel/src/debug_code/*.h"
  "Core/SoarKernel/src/decision_process/*.h"
  "Core/SoarKernel/src/episodic_memory/*.h"
  "Core/SoarKernel/src/explanation_based_chunking/*.h"
  "Core/SoarKernel/src/explanation_memory/*.h"
  "Core/SoarKernel/src/interface/*.h"
  "Core/SoarKernel/src/output_manager/*.h"
  "Core/SoarKernel/src/parsing/*.h"
  "Core/SoarKernel/src/reinforcement_learning/*.h"
  "Core/SoarKernel/src/semantic_memory/*.h"
  "Core/SoarKernel/src/shared/*.h"
  "Core/SoarKernel/src/soar_representation/*.h"
  "Core/SoarKernel/src/visualizer/*.h"
  "Core/shared/Export.h"
  "Core/shared/build_time_date.h"
  "Core/shared/soarversion.h"
  "Core/shared/tokenizer.h"
  "Core/shared/soar_TraceNames.h"
  "Core/shared/portability.h"
  "Core/shared/portability_posix.h"
)

# ASAN only works in debug mode.
option(ASAN "Enable AddressSanitizer" OFF)
option(SVS "SVS Enabled" OFF)

if(NOT SVS)
  message(STATUS "Soar: Disabling SVS")
  add_compile_definitions(NO_SVS)
else()
	if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows" OR ${CMAKE_SYSTEM_NAME} STREQUAL "MSYS")
		set(PLATFORM_SPEC windows)
	elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Darwin" OR ${CMAKE_SYSTEM_NAME} STREQUAL "iOS")
		set(PLATFORM_SPEC osx)
	else()
		set(PLATFORM_SPEC posix)
	endif()

	file(GLOB_RECURSE svs_deps_sources "Core/SVS/ccd/*.c")
	add_library(svs_deps_objs OBJECT ${svs_deps_sources})
	target_include_directories(svs_deps_objs PRIVATE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SVS/ccd>)
	set_property(TARGET svs_deps_objs
				 PROPERTY POSITION_INDEPENDENT_CODE True
				 PROPERTY C_STANDARD 11
				 PROPERTY LINKER_LANGUAGE C)
				 
	list(APPEND SOURCES Core/SVS/SVS.cxx Core/SVS/src/${PLATFORM_SPEC}/platform_specific.cpp)	
endif()

# Add executable/library target
add_library(${PROJECT_NAME}_lib  SHARED  ${SOURCES})
target_include_directories(${PROJECT_NAME}_lib PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/CLI/src>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/ClientSML/src>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/ConnectionSML/src>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/ElementXML/src>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/KernelSML/src>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SVS/src>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/debug_code>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/decision_process>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/episodic_memory>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/explanation_based_chunking>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/explanation_memory>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/interface>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/output_manager>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/parsing>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/reinforcement_learning>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/semantic_memory>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/shared>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/soar_representation>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SoarKernel/src/visualizer>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/shared>
  $<INSTALL_INTERFACE:include/soar>
)

set(LIB_DEPS $<TARGET_OBJECTS:SQLite3_objs>)
if(SVS)
 target_include_directories(${PROJECT_NAME}_lib PRIVATE 
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SVS/eigen>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SVS/ccd>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/SVS/src/${PLATFORM_SPEC}>)
 list(APPEND LIB_DEPS $<TARGET_OBJECTS:svs_deps_objs>)
endif()

target_link_libraries(${PROJECT_NAME}_lib PUBLIC ${LIB_DEPS})

set_target_properties(${PROJECT_NAME}_lib
  PROPERTIES
  PUBLIC_HEADER "${HEADERS}"
  PROPERTY POSITION_INDEPENDENT_CODE True
  PROPERTY C_STANDARD 11
  PROPERTY CXX_STANDARD 17
  PROPERTY LINKER_LANGUAGE CXX)

set(CXX_STANDARD_REQUIRED True)

# Set the C++ standard for this and all downstream targets
target_compile_features(${PROJECT_NAME}_lib PUBLIC
  cxx_std_17
)

if($<CONFIG:DEBUG> OR $<CONFIG:RELWITHDEBINFO>)
  target_compile_options(${PROJECT_NAME}_lib PRIVATE $<$<CXX_COMPILER_ID:Clang,AppleClang,GNU>: -Wall -Wpedantic> -Wa,-mbig-obj)

  target_compile_options(${PROJECT_NAME}_lib PRIVATE $<$<CXX_COMPILER_ID:MSVC>: /Wall /permissive> /bigobj)

  if(NOT ${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    if(ASAN)
      message(STATUS "Soar: Enabling AddressSanitizer")
      add_compile_options(-fsanitize=address -fno-omit-frame-pointer)
      add_link_options(-fsanitize=address)
    endif()

    find_program(CLANG_TIDY_EXE NAMES clang-tidy)

    if(CLANG_TIDY_EXE)
      set(CMAKE_CXX_CLANG_TIDY ${CLANG_TIDY_EXE} -checks=-*,readability-*)
    else()
      message(WARNING "clang-tidy not found!")
    endif()
  else()
    message(WARNING "ASAN Debug flags for Windows not set up.")
  endif()
endif()


if (MSVC)
    # warning level 4
    target_compile_options(${PROJECT_NAME}_lib PRIVATE /D_USRDLL /D_CRT_SECURE_NO_DEPRECATE)
endif()


include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/soarConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion)

# This only sets up the install structure, but no installation happens yet.
install(TARGETS ${PROJECT_NAME}_lib
  EXPORT SoarTargets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/soar
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/soar
)

install(FILES ${HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/soar)

install(EXPORT SoarTargets
  FILE SoarTargets.cmake
  NAMESPACE soar::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/soar
)

include(CMakePackageConfigHelpers)
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/soarConfig.cmake"
  INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/soar"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/soarConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/soarConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/soar
)

export(EXPORT SoarTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/SoarTargets.cmake"
)

add_library(soar::soar_lib ALIAS ${PROJECT_NAME}_lib)

if(CLI)
  message(STATUS "Soar: Building CLI")
  add_subdirectory(SoarCLI)
endif()

if(BUILD_TESTS)
  message(STATUS "Soar: Building tests")
  include(CTest)
  add_subdirectory(PerformanceTests)
  add_subdirectory(UnitTests)
endif()

# Include CPack module
include(CPack)

# Set CPack variables
set(CPACK_PACKAGE_NAME "${PROJECT_NAME}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Soar is a cognitive architecture for developing systems that exhibit intelligent behavior.")
set(CPACK_PACKAGE_CONTACT "soar-admin@umich.edu")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-Source")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.md")
set(CPACK_PACKAGE_VENDOR "University of Michigan Soar Group")