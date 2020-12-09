# Sets the minimum version of CMake required to build the native
# library. You should either keep the default value or only pass a
# value of 3.4.0 or lower.
# by huzongyao

cmake_minimum_required(VERSION 3.0)

option (BUILD_SHARED_LIB "compatible format" ON )

include("../cmake/source_group_by_dir.cmake")


# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds it for you.
# Gradle automatically packages shared libraries with your APK.
file(GLOB_RECURSE LUASQL_SRCS 
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luasql.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ls_mysql.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luasql.h
    # ${CMAKE_CURRENT_SOURCE_DIR}/mysql/libmariadb/libmariadb/**.c
)
# message(stats "${LUASQL_SRCS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -p -g -O2 -Wl,-rpath=./")

message("CMAKE_SOURCE_DIR=${CMAKE_SOURCE_DIR}")

# add_subdirectory(mysql/libmariadb/libmariadb)

include_directories(
    ${CMAKE_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/
    ${CMAKE_CURRENT_SOURCE_DIR}/../lua
    /usr/local/include/mysql
    # /Volumes/Data/a3/tools/xlua/build/mysql/libmariadb/include
    # # ${CMAKE_BINARY_DIR}/libmariadb/include
    # /Volumes/Data/hxyx/s/dep/usr/aarch64/include
    # /Volumes/Data/hxyx/s/dep/usr/aarch64/include/mysql
    # # /Volumes/Data/a3/tools/xlua/build/mysql/build.Android.armeabi-v7a/libmariadb/include
)

link_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/libs/$ENV{ABI}
    ${CMAKE_BINARY_DIR}/bin
    # ${CMAKE_BINARY_DIR}/lib
    /usr/local/lib/
)

if (BUILD_SHARED_LIB)
    add_library(luasql SHARED ${LUASQL_SRCS})

    # find_library(lua lua)
    target_link_libraries(luasql mariadb liblua )

    SET_TARGET_PROPERTIES(luasql PROPERTIES 
        LINK_FLAGS "-undefined dynamic_lookup" #
        # SUFFIX ".so"
        # OUTPUT_NAME "luasql"
    )

    # target_link_libraries(luasql z)

    # find_library(xlua xlua)
    # target_link_libraries(luasql z)

    ## https://cmake.org/cmake/help/git-stage/command/target_link_options.html
    # target_link_options(luasql 
    #   INTERFACE lua
    # )
else()
    add_library(luasql STATIC ${LUASQL_SRCS})
endif(BUILD_SHARED_LIB)
source_group_by_dir(${CMAKE_CURRENT_SOURCE_DIR} LUASQL_SRCS)

if(ANDROID)
    message("ANDROID_ABI=$ENV{ANDROID_ABI}")
    target_link_libraries(luasql
        # crypto
        # ssl
        iconv
        z
    )
endif(ANDROID)



# if (APPLE AND BUILD_SHARED_LIB)
# 	find_library(ObjC objc)
# 	find_library(CoreFoundation CoreFoundation)
# 	find_library(IOKit IOKit)
#     target_link_libraries(luasql 
#         ${CoreFoundation} 
#         ${IOKit} 
#         ${ObjC}
#     )
    
#     if(LUASQL_EXECUTABLE)
#         target_link_libraries(luasql 
#             ${CoreFoundation} 
#             ${IOKit} 
#             ${ObjC}
#         )
#     endif(LUASQL_EXECUTABLE)
# endif()
