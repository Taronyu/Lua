cmake_minimum_required(VERSION 3.2 FATAL_ERROR)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type: Debug or Release" FORCE)
endif()

if(CMAKE_CONFIGURATION_TYPES)
	set(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "Limited configurartions" FORCE)
	mark_as_advanced(CMAKE_CONFIGURATION_TYPES)
endif()

project(lua VERSION 5.3.2 LANGUAGES NONE)

option(LUA_BUILD_LUAC "Build Lua compiler" ON)
option(LUA_BUILD_CLI "Build Lua CLI interpreter" ON)
option(LUA_BUILD_CXX "Compile Lua as C++ project" OFF)

if(LUA_BUILD_CXX)
	enable_language(CXX)
else()
	enable_language(C)
endif()

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMake")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_DEBUG_POSTFIX "-d")

if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
	set(LUA_OS_LINUX 1)
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
	set(LUA_OS_WINDOWS 1)
endif()

set(INSTALL_MISC_DIR share/lua)

SET(LUA_SRC
	src/lapi.c src/lcode.c src/lctype.c src/ldebug.c src/ldo.c src/ldump.c
	src/lfunc.c src/lgc.c src/llex.c src/lmem.c src/lobject.c src/lopcodes.c
	src/lparser.c src/lstate.c src/lstring.c src/ltable.c src/ltm.c
	src/lundump.c src/lvm.c src/lzio.c src/lauxlib.c src/lbaselib.c
	src/lbitlib.c src/lcorolib.c src/ldblib.c src/liolib.c src/lmathlib.c
	src/loslib.c src/lstrlib.c src/ltablib.c src/lutf8lib.c src/loadlib.c
	src/linit.c)

add_library(lua ${LUA_SRC})

target_compile_definitions(lua PUBLIC LUA_COMPAT_5_2)
set_target_properties(lua PROPERTIES DEFINE_SYMBOL LUA_EXPORTS)

if(CMAKE_COMPILER_IS_GNUCXX)
	target_compile_options(lua PRIVATE -Wall -Wextra)
	set_target_properties(lua PROPERTIES LINK_FLAGS "-Wl,-E")
endif()

if(CMAKE_COMPILER_IS_GNUCC)
	target_compile_options(lua PRIVATE -std=gnu99 -Wall -Wextra)
	set_target_properties(lua PROPERTIES LINK_FLAGS "-Wl,-E")
endif()

if(LUA_OS_LINUX)
	target_compile_definitions(lua PUBLIC LUA_USE_LINUX)
	target_link_libraries(lua PRIVATE m dl readline)
elseif(LUA_OS_WINDOWS)
	# LUA_USE_WINDOWS is defined in luaconf.h
	target_compile_definitions(lua PUBLIC LUA_BUILD_AS_DLL)
endif()

install(TARGETS lua
	RUNTIME DESTINATION bin COMPONENT bin
	LIBRARY DESTINATION lib${LIB_SUFFIX} COMPONENT bin
	ARCHIVE DESTINATION lib${LIB_SUFFIX} COMPONENT devel)

install(FILES src/lua.h src/luaconf.h src/lualib.h src/lauxlib.h src/lua.hpp
	DESTINATION include/lua
	COMPONENT devel)

if(LUA_BUILD_LUAC)
	add_executable(luac ${LUA_SRC} src/luac.c)

	if(LUA_OS_LINUX)
		target_link_libraries(luac PRIVATE m dl readline)
	endif()

	install(TARGETS luac
		RUNTIME DESTINATION bin COMPONENT bin
		LIBRARY DESTINATION lib${LIB_SUFFIX} COMPONENT bin
		ARCHIVE DESTINATION lib${LIB_SUFFIX} COMPONENT devel)
endif()

if(LUA_BUILD_CLI)
	add_executable(luacli src/lua.c)
	target_link_libraries(luacli PRIVATE lua readline)
	set_target_properties(luacli PROPERTIES OUTPUT_NAME lua)

	install(TARGETS luacli
		RUNTIME DESTINATION bin COMPONENT bin
		LIBRARY DESTINATION lib${LIB_SUFFIX} COMPONENT bin
		ARCHIVE DESTINATION lib${LIB_SUFFIX} COMPONENT devel)
endif()

if(LUA_BUILD_CXX)
	set_source_files_properties(${LUA_SRC} src/lua.c src/luac.c
		PROPERTIES LANGUAGE CXX)
endif()

