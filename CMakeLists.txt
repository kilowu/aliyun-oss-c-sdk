PROJECT(oss_c_sdk)

CMAKE_MINIMUM_REQUIRED(VERSION 2.6)  

set(CMAKE_VERSION 3.0.0)

# default C / CXX flags
set(CMAKE_C_FLAGS " -g -ggdb -O0 -Wall -fpic -fPIC -D_LARGEFILE64_SOURCE")
set(CMAKE_CXX_FLAGS " -g -ggdb -O0 -Wall -fpic -fPIC -D_LARGEFILE64_SOURCE")

set(CMAKE_C_FLAGS_DEBUG " -g -ggdb -O0 -Wall -fpic -fPIC -D_LARGEFILE64_SOURCE")
set(CMAKE_CXX_FLAGS_DEBUG " -g -ggdb -O0 -Wall -fpic -fPIC -D_LARGEFILE64_SOURCE")

set(CMAKE_C_FLAGS_RELEASE " -O3 -Wall -fpic -fPIC -D_LARGEFILE64_SOURCE")
set(CMAKE_CXX_FLAGS_RELEASE " -O3 -Wall -fpic -fPIC -D_LARGEFILE64_SOURCE")

set(CMAKE_C_FLAGS_MINSIZEREF " -Os -Wall  -fpic -fPIC -D_LARGEFILE64_SOURCE")
set(CMAKE_CXX_FLAGS_MINSIZEREF " -Os -Wall -fpic -fPIC -D_LARGEFILE64_SOURCE")

set(CMAKE_C_FLAGS_RELWITHDEBINFO " -O2 -Wall -fpic -fPIC -D_LARGEFILE64_SOURCE")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO " -O2 -Wall -fpic -fPIC -D_LARGEFILE64_SOURCE")

set(CMAKE_C_FLAGS_COVERAGE " ${CMAKE_C_FLAGS_DEBUG} -fprofile-arcs -ftest-coverage")
set(CMAKE_CXX_FLAGS_COVERAGE " ${CMAKE_C_FLAGS_DEBUG} -fprofile-arcs -ftest-coverag")

set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/build/${CMAKE_BUILD_TYPE}/lib)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/build/${CMAKE_BUILD_TYPE}/bin)
set(CMAKE_C_OUTPUT_EXTENSION_REPLACE 1)
set(CMAKE_CXX_OUTPUT_EXTENSION_REPLACE 1)

FIND_PROGRAM(APR_CONFIG_BIN NAMES apr-config apr-1-config PATHS /usr/bin /usr/local/bin /usr/local/apr/bin/)
FIND_PROGRAM(APU_CONFIG_BIN NAMES apu-config apu-1-config PATHS /usr/bin /usr/local/bin /usr/local/apr/bin/)

IF (APR_CONFIG_BIN)
    EXECUTE_PROCESS(
        COMMAND ${APR_CONFIG_BIN} --includedir
        OUTPUT_VARIABLE APR_INCLUDE_DIR
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    EXECUTE_PROCESS(
        COMMAND ${APR_CONFIG_BIN} --cflags
        OUTPUT_VARIABLE APR_C_FLAGS
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    EXECUTE_PROCESS(
        COMMAND ${APR_CONFIG_BIN} --link-ld
        OUTPUT_VARIABLE APR_LIBRARIES
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
ELSE()
    MESSAGE(FATAL_ERROR "Could not find apr-config/apr-1-config")
ENDIF()

IF (APU_CONFIG_BIN)
    EXECUTE_PROCESS(
        COMMAND ${APU_CONFIG_BIN} --includedir
        OUTPUT_VARIABLE APR_UTIL_INCLUDE_DIR
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    EXECUTE_PROCESS(
        COMMAND ${APU_CONFIG_BIN} --cflags
        OUTPUT_VARIABLE APU_C_FLAGS
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    EXECUTE_PROCESS(
        COMMAND ${APU_CONFIG_BIN} --link-ld
        OUTPUT_VARIABLE APU_LIBRARIES
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
ELSE()
    MESSAGE(FATAL_ERROR "Could not find apu-config/apu-1-config")
ENDIF()

#curl-config
FIND_PROGRAM(CURL_CONFIG_BIN NAMES curl-config)
  
IF (CURL_CONFIG_BIN)
    EXECUTE_PROCESS(
        COMMAND ${CURL_CONFIG_BIN} --libs
        OUTPUT_VARIABLE CURL_LIBRARIES
        OUTPUT_STRIP_TRAILING_WHITESPACE
        )
ELSE()
    MESSAGE(FATAL_ERROR "Could not find curl-config")
ENDIF()
  
# Compile and link lib_oss_c_sdk
include_directories(${APR_INCLUDE_DIR})
include_directories(${APR_UTIL_INCLUDE_DIR})
include_directories(${MINIXML_INCLUDE_DIR})
include_directories(${CURL_INCLUDE_DIR})

aux_source_directory(oss_c_sdk SRC_LIST)

add_library(${CMAKE_PROJECT_NAME} SHARED ${SRC_LIST})
add_library(${CMAKE_PROJECT_NAME}_static STATIC ${SRC_LIST})

set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES VERSION ${CMAKE_VERSION}  SOVERSION ${CMAKE_VERSION})

INSTALL(TARGETS ${CMAKE_PROJECT_NAME} LIBRARY DESTINATION lib)

INSTALL(TARGETS ${CMAKE_PROJECT_NAME}_static ARCHIVE DESTINATION lib)

INSTALL(FILES
  oss_c_sdk/aos_buf.h
  oss_c_sdk/aos_define.h
  oss_c_sdk/aos_fstack.h
  oss_c_sdk/aos_http_io.h
  oss_c_sdk/aos_list.h
  oss_c_sdk/aos_log.h
  oss_c_sdk/aos_status.h
  oss_c_sdk/aos_string.h
  oss_c_sdk/aos_transport.h
  oss_c_sdk/aos_util.h
  oss_c_sdk/oss_api.h
  oss_c_sdk/oss_auth.h
  oss_c_sdk/oss_define.h
  oss_c_sdk/oss_util.h
  oss_c_sdk/oss_xml.h
  DESTINATION include/oss_c_sdk)

add_subdirectory(oss_c_sdk_sample)
add_subdirectory(oss_c_sdk_test)
