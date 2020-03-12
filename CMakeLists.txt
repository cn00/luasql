# Sets the minimum version of CMake required to build the native
# library. You should either keep the default value or only pass a
# value of 3.4.0 or lower.
# by huzongyao

cmake_minimum_required(VERSION 3.0)

option (BUILD_SHARED_LIB "compatible format" ON )

macro(source_group_by_dir proj_dir source_files)
    if(MSVC OR APPLE)
        get_filename_component(sgbd_cur_dir ${proj_dir} ABSOLUTE)
        foreach(sgbd_file ${${source_files}})
            get_filename_component(sgbd_abs_file ${sgbd_file} ABSOLUTE)
            file(RELATIVE_PATH sgbd_fpath ${sgbd_cur_dir} ${sgbd_abs_file})
            string(REGEX REPLACE "\(.*\)/.*" \\1 sgbd_group_name ${sgbd_fpath})
            string(COMPARE EQUAL ${sgbd_fpath} ${sgbd_group_name} sgbd_nogroup)
            string(REPLACE "/" "\\" sgbd_group_name ${sgbd_group_name})
            if(sgbd_nogroup)
                set(sgbd_group_name "\\")
            endif(sgbd_nogroup)
            source_group(${sgbd_group_name} FILES ${sgbd_file})
        endforeach(sgbd_file)
    endif(MSVC OR APPLE)
endmacro(source_group_by_dir)


# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds it for you.
# Gradle automatically packages shared libraries with your APK.
file(GLOB_RECURSE LUASQL_SRCS 
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luasql.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ls_mysql.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luasql.h
)
# message(stats "${LUASQL_SRCS}")

message("CMAKE_SOURCE_DIR=${CMAKE_SOURCE_DIR}")

include_directories(
    ${CMAKE_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/
    ${CMAKE_CURRENT_SOURCE_DIR}/../lua-5.3.5/src
    /usr/local/include/mysql
    ${CMAKE_CURRENT_SOURCE_DIR}/../build_osx
)


if (BUILD_SHARED_LIB)
    add_library(luasql SHARED ${LUASQL_SRCS})

    SET_TARGET_PROPERTIES(luasql PROPERTIES 
        LINK_FLAGS "-undefined dynamic_lookup" #
        # SUFFIX ".so"
        # OUTPUT_NAME "luasql"
    )

    find_library(mysqlclient mysqlclient)
    target_link_libraries(luasql ${mysqlclient})

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
