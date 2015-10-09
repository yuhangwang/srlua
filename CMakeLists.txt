# Copyright (C) 2012 LuaDist.
# Created by Peter Drahoš, Peter Kapec
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.
# Modified by: Yuhang Wang
# Date: 10/08/2015

project ( srlua C )
cmake_minimum_required ( VERSION 2.8 )
include ( cmake/dist.cmake )

include_directories("/usr/lib64") # I have glibc-static insall to /usr/lib64
include_directories(${LUA_INCLUDE_DIR})

find_package ( Lua REQUIRED )
find_library(LIB_DL libdl.a)
message("lua lib" ${LUA_LIBRARIES})
message(${LIB_DL})

# note: when using the "-static" flag, 
# you must APPEND (not prepend) "-ldl" to the end of gcc command 
# to avoid the following error:
# loadlib.c:(.text+0x4b2): undefined reference to `dlsym'
# loadlib.c:(.text+0x4f9): undefined reference to `dlerror'
# loadlib.c:(.text+0x526): undefined reference to `dlopen'
# loadlib.c:(.text+0x59d): undefined reference to `dlerror'
# /Scr/scr-test-steven/install/lua/5.3.1/lib/liblua.a(loadlib.o): In function `gctm':
# loadlib.c:(.text+0x731): undefined reference to `dlclose'
# collect2: error: ld returned 1 exit status
# make[2]: *** [srlua] Error 1
# make[1]: *** [CMakeFiles/srlua.dir/all] Error 2
# make: *** [all] Error 2
# The equivalent command is:
# gcc -static CMakeFiles/srlua.dir/srlua.c.o \
# -o srlua -rdynamic /Scr/scr-test-steven/install/lua/5.3.1/lib/liblua.a -lm -ldl
# The trick in CMake is to use target_link_libraries(TAREGET LIBRARY_NAME)
# CMake will add -lLIBRARY_NAME to the end of the compilation command 
# 
# note 2: use "make VERBOSE=1" to see the command used by CMake 

message("CMAKE_EXE_LINKER_FLAGS " ${CMAKE_EXE_LINKER_FLAGS})
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")

add_executable(glue glue.c glue.rc)
add_executable(srlua srlua.c srlua.rc)

set_target_properties(srlua PROPERTIES LINK_FLAGS "-static")

target_link_libraries(glue ${LUA_LIBRARIES})
target_link_libraries(srlua ${LUA_LIBRARIES} dl)


# No console version on windows
if (WIN32)
  add_executable(wsrlua WIN32 wmain.c srlua.c srlua.rc)
  target_link_libraries(wsrlua ${LUA_LIBRARIES})
  install_executable(wsrlua)
elseif ( APPLE )
  set ( CMAKE_EXE_LINKER_FLAGS "-pagezero_size 10000 -image_base 100000000 -image_base 7fff04c4a000" )
endif()

install_executable(glue srlua)
install_data(README.md)
install_test(test.lua)
