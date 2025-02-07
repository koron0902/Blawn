cmake_minimum_required(VERSION 3.13)

project(blawn C CXX)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

# LLVM-Config.cmakeを探すためのやつ
file(GLOB_RECURSE LLVM_CMAKE /usr/lib*/llvm?.0/lib/cmake/llvm/LLVM-Config.cmake)
foreach(CMAKE IN LISTS LLVM_CMAKE)
		include(${CMAKE})
endforeach()
message(STATUS "LLVM module found on ${LLVM_CMAKE}")

# Disable in-source build
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
set(CMAKE_DISABLE_SOURCE_CHANGES  ON)

# Set default build type
if(NOT CMAKE_BUILD_TYPE)
		set(CMAKE_BUILD_TYPE Release)
endif()
	message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

# Set compiler flags
set(CMAKE_CXX_EXTENSIONS  OFF)
set(CMAKE_CXX_STANDARD    17)
set(CXX_STANDARD_REQUIRED ON)

set(CMAKE_CXX_FLAGS         "${CMAKE_CXX_FLAGS}         -Wall -Wextra -pedantic")
set(CMAKE_CXX_FLAGS_DEBUG   "${CMAKE_CXX_FLAGS_DEBUG}   -g -DDEBUG")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2 -march=native -DNDEBUG")
set(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS}  -Wl,-O1,--sort-common,--as-needed,-z,relro")

if(CMAKE_GENERATOR STREQUAL "Ninja")
		if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
				set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdiagnostics-color=always")
		elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
				set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fcolor-diagnostics")
		 endif()
endif()

# Sanitizers
if(ENABLE_SANITIZERS)
		if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
				message(STATUS "Enabling GCC's address sanitizer")
				set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address")
		elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
				message(STATUS "Enabling Clang's address sanitizer")
				set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer")
		endif()
endif()


include_directories(${PROJECT_SOURCE_DIR}/src)

find_package(BISON REQUIRED)
find_package(FLEX REQUIRED)
find_package(LLVM)

add_subdirectory(src)
