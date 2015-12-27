cmake_minimum_required(VERSION 2.8)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type: Debug or Release" FORCE)
endif()

if(CMAKE_CONFIGURATION_TYPES)
	set(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "Limited configurartions" FORCE)
	mark_as_advanced(CMAKE_CONFIGURATION_TYPES)
endif()

project(lua NONE)

# project version
set(VERSION_MAJOR 5)
set(VERSION_MINOR 3)
set(VERSION_PATCH 2)

option(LUA_BUILD_LUAC "Build Lua compiler" ON)
option(LUA_BUILD_CLI "Build Lua CLI interpreter" ON)
option(LUA_BUILD_CXX "Compile Lua as C++ project" OFF)

# enable project folders
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMake")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_DEBUG_POSTFIX "-d")

# OS type
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
	set(LUA_OS_LINUX 1)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
	set(LUA_OS_WINDOWS 1)
endif()

if(LUA_BUILD_CXX)
	enable_language(CXX)
else()
	enable_language(C)
endif()

if(CMAKE_COMPILER_IS_GNUCXX)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
	#set(CMAKE_CXX_FLAGS_DEBUG "-O0 -g -DDEBUG=1")
	#set(CMAKE_CXX_FLAGS_RELEASE "-O2 -DNDEBUG=1")
	set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-E")
endif()

if(CMAKE_COMPILER_IS_GNUCC)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=gnu99 -Wall -Wextra")
	#set(CMAKE_C_FLAGS_DEBUG "-O0 -g -DDEBUG=1")
	#set(CMAKE_C_FLAGS_RELEASE "-O2 -DNDEBUG=1")
	set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-E")
endif()

set(INSTALL_MISC_DIR share/lua)

add_subdirectory(src)

