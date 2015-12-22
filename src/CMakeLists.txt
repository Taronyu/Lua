SET(LUA_SRC
	lapi.c lcode.c lctype.c ldebug.c ldo.c ldump.c lfunc.c lgc.c llex.c lmem.c
	lobject.c lopcodes.c lparser.c lstate.c lstring.c ltable.c ltm.c lundump.c
	lvm.c lzio.c lauxlib.c lbaselib.c lbitlib.c lcorolib.c ldblib.c liolib.c
	lmathlib.c loslib.c lstrlib.c ltablib.c lutf8lib.c loadlib.c linit.c)

add_library(lua ${LUA_SRC})

set_target_properties(lua PROPERTIES SOVERSION ${VERSION_MAJOR}.${VERSION_MINOR})
set_target_properties(lua PROPERTIES VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH})
set_target_properties(lua PROPERTIES DEFINE_SYMBOL LUA_BUILD_AS_DLL)

if(NOT BUILD_SHARED_LIBS)
	set_target_properties(lua PROPERTIES POSITION_INDEPENDENT_CODE TRUE)
endif()

add_definitions(-DLUA_COMPAT_5_2)

if(LUA_OS_LINUX)
	add_definitions(-DLUA_USE_LINUX)
	target_link_libraries(lua m dl readline)
elseif(LUA_OS_WINDOWS)
	add_definitions(-DLUA_USE_WINDOWS)
endif()

install(TARGETS lua
	RUNTIME DESTINATION bin COMPONENT bin
	LIBRARY DESTINATION lib${LIB_SUFFIX} COMPONENT bin
	ARCHIVE DESTINATION lib${LIB_SUFFIX} COMPONENT devel)

install(FILES lua.h luaconf.h lualib.h lauxlib.h lua.hpp
	DESTINATION include/lua
	COMPONENT devel)

if(LUA_BUILD_LUAC)
	add_executable(luac ${LUA_SRC} luac.c)

	if(LUA_OS_LINUX)
		target_link_libraries(luac m dl readline)
	endif()

	install(TARGETS luac
		RUNTIME DESTINATION bin COMPONENT bin
		LIBRARY DESTINATION lib${LIB_SUFFIX} COMPONENT bin
		ARCHIVE DESTINATION lib${LIB_SUFFIX} COMPONENT devel)
endif()

if(LUA_BUILD_CLI)
	add_executable(luacli lua.c)

	target_link_libraries(luacli lua)

	set_target_properties(luacli PROPERTIES OUTPUT_NAME lua)

	install(TARGETS luacli
		RUNTIME DESTINATION bin COMPONENT bin
		LIBRARY DESTINATION lib${LIB_SUFFIX} COMPONENT bin
		ARCHIVE DESTINATION lib${LIB_SUFFIX} COMPONENT devel)
endif()
