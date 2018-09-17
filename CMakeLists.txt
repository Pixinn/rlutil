# This is an header only library
# On Windows, sets the CMAKE_PACKAGES_DIR environment variable to indicate where to install

cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

project(rlutil VERSION 1.0)

set(LIB_NAME ${PROJECT_NAME})

## HEADER LIBRARY
include(GNUInstallDirs)
message("DBG: " ${CMAKE_INSTALL_PREFIX})
add_library(${LIB_NAME} INTERFACE)
target_include_directories(	${LIB_NAME} INTERFACE 
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>)
# Package
if(CMAKE_SYSTEM MATCHES "Windows") # Where to install packages. CMAKE_INSTALL_PREFIX would default to 
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    if("$ENV{CMAKE_PACKAGES_DIR}" STREQUAL "")
        message(FATAL_ERROR "\n!!!!!!!!! FATAL ERROR !!!!!!!!!\nPlease define CMAKE_INSTALL_PREFIX or set a CMAKE_PACKAGES_DIR environment variable.\nDelete CMakeCache.txt then proceed.\n!!!!!!!!!")
    else()
        set(CMAKE_INSTALL_PREFIX $ENV{CMAKE_PACKAGES_DIR} CACHE PATH "..." FORCE) 
    endif()
endif()
endif(CMAKE_SYSTEM MATCHES "Windows")

install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/include/ DESTINATION  ${CMAKE_INSTALL_PREFIX}/include/${LIB_NAME})   # this one will copy the whole include dir
install(    # this one will copy the built artifacts
            TARGETS ${LIB_NAME}
			EXPORT ${LIB_NAME}-targets
			PUBLIC_HEADER DESTINATION   ${CMAKE_INSTALL_INCLUDEDIR}/${LIB_NAME} # defined with set_target_properties()
)
install(     # this one will copy the .cmake files
            EXPORT ${LIB_NAME}-targets
            FILE ${LIB_NAME}-config.cmake
            DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${LIB_NAME}
		)


## TESTS
set(COMPILE_OPTIONS_MSVC "/W4 /MP /WX")
set(COMPILE_OPTIONS_UNIX  "-Wall -Wextra  -Werror -Wno-missing-braces") #"missing-braces" only emitted by 

add_executable("a-test" ${CMAKE_CURRENT_SOURCE_DIR}/test.cpp)
target_include_directories("a-test" PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)

add_executable("example-c" ${CMAKE_CURRENT_SOURCE_DIR}/example.c)
target_include_directories("example-c" PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)

if(MSVC)
    set_target_properties("a-test" "example-c" PROPERTIES COMPILE_FLAGS ${COMPILE_OPTIONS_MSVC})
else()
    set_target_properties("a-test" "example-c" PROPERTIES COMPILE_FLAGS ${COMPILE_OPTIONS_UNIX})
endif()
