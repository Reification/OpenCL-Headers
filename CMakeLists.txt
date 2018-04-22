cmake_minimum_required(VERSION 3.11)
project(OpenCL-Loader)

set(CMAKE_CONFIGURATION_TYPES "Debug;Release")

file(GLOB OPENCL_VERSIONS LIST_DIRECTORIES true RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "${CMAKE_CURRENT_SOURCE_DIR}/opencl*")
list(REMOVE_ITEM OPENCL_VERSIONS opencl_loader)

set(OPENCL_VERSION "opencl12" CACHE STRING "OpenCL API Version")
set_property(CACHE OPENCL_VERSION PROPERTY STRINGS ${OPENCL_VERSIONS})

if(NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/opencl_loader/${OPENCL_VERSION})
  execute_process ( COMMAND bash ${CMAKE_CURRENT_SOURCE_DIR}/generator/generate_loader.sh ${OPENCL_VERSION}
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/ )
else()
  message(STATUS "OpenCL API Loader for ${OPENCL_VERSION} exists.")
endif()

set(API_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/opencl_loader/${OPENCL_VERSION}/include")

file(GLOB API_HEADERS "${API_INCLUDE_DIR}/CL/*.h")
file(GLOB LIB_SRC "opencl_loader/${OPENCL_VERSION}/src/*.c")

add_library(${PROJECT_NAME} STATIC ${LIB_SRC} ${API_HEADERS})

source_group("CL" FILES ${API_HEADERS})

target_include_directories(
 ${PROJECT_NAME}
 PRIVATE
 ${API_INCLUDE_DIR}
)

install(FILES $<TARGET_FILE:${PROJECT_NAME}> DESTINATION lib)
install(DIRECTORY ${API_INCLUDE_DIR}/CL DESTINATION include/)