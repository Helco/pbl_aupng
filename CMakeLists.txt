cmake_minimum_required (VERSION 3.10 FATAL_ERROR)
project(aupng)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

###################################################################
# googletest
###################################################################

configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif()

# Prevent overriding the parent project's compiler/linker
# settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# Add googletest directly to our build. This defines
# the gtest and gtest_main targets.
add_subdirectory(${CMAKE_BINARY_DIR}/googletest-src
                 ${CMAKE_BINARY_DIR}/googletest-build
                 EXCLUDE_FROM_ALL)

# The gtest/gtest_main targets carry header search path
# dependencies automatically when using CMake 2.8.11 or
# later. Otherwise we have to add them here ourselves.
if (CMAKE_VERSION VERSION_LESS 2.8.11)
  include_directories("${gtest_SOURCE_DIR}/include")
endif()
enable_testing()
include(GoogleTest)

###################################################################
# aupng
###################################################################

add_library(aupng
    src/upng.h
    src/upng_internal.h
    src/upng.c
    src/upng_inflate.c
    src/upng_decode.c
    src/upng_config.h
)
target_include_directories(aupng
    PUBLIC src
)

###################################################################
# test_aupng
###################################################################

add_executable(test_aupng
    test/DebugAllocator.hpp
    test/DebugAllocator.cpp
    test/test_common.hpp

    test/test_single_picture.cpp
    test/test_memory.cpp
    test/test_multiple_frames.cpp
)
target_link_libraries(test_aupng
    PRIVATE aupng
    PRIVATE gtest_main
)
gtest_discover_tests(test_aupng
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)
