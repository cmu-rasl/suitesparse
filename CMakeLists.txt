cmake_minimum_required(VERSION 2.8.3)
project(suitesparse)

set(CMAKE_BUILD_TYPE Release)

find_package(catkin REQUIRED)

include(ExternalProject)

set(suitesparse_PREFIX "${CMAKE_CURRENT_BINARY_DIR}/suitesparse")
set(suitesparse_SOURCE_DIR "${suitesparse_PREFIX}/src/suitesparse")
set(suitesparse_BUILD_DIR "${suitesparse_SOURCE_DIR}_src-build/SuiteSparse")
execute_process(COMMAND ${CMAKE_COMMAND} -E make_directory ${suitesparse_PREFIX}/src)

set(VERSION 4.5.3)

catkin_package(
  DEPENDS
  CATKIN_DEPENDS
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  )

ExternalProject_Add(suitesparse_src
  PREFIX ${suitesparse_PREFIX}
  SOURCE_DIR ${suitesparse_SOURCE_DIR}
  DOWNLOAD_COMMAND wget --retry-connrefused --waitretry=1 --timeout=40 --tries 3 http://faculty.cse.tamu.edu/davis/SuiteSparse/SuiteSparse-${VERSION}.tar.gz
  PATCH_COMMAND tar -xzf ../SuiteSparse-${VERSION}.tar.gz && rm -rf ../suitesparse_src-build/SuiteSparse && sed -i.bu "s/\\/usr\\/local\\/lib/..\\/lib/g" SuiteSparse/SuiteSparse_config/SuiteSparse_config.mk && sed -i.bu "s/\\/usr\\/local\\/include/..\\/include/g" SuiteSparse/SuiteSparse_config/SuiteSparse_config.mk && mv SuiteSparse ../suitesparse_src-build/
  CONFIGURE_COMMAND ""
  BUILD_COMMAND export CFLAGS=-DNTIMER && cd SuiteSparse && make library -j8 -l8
  INSTALL_COMMAND cd SuiteSparse && mkdir -p lib && mkdir -p include && make install)

set(suitesparse_INCLUDE_DIRS "${CMAKE_INSTALL_PREFIX}/include")

add_library(${PROJECT_NAME} src/export_lib_hack.cc)
FILE(GLOB ALL_LIBRARIES ${suitesparse_BUILD_DIR}/lib/*.a)
target_link_libraries(${PROJECT_NAME}
  ${ALL_LIBRARIES})
add_dependencies(${PROJECT_NAME} suitesparse_src)

install(DIRECTORY ${suitesparse_BUILD_DIR}/include/
  DESTINATION "${CMAKE_INSTALL_PREFIX}/include/${PROJECT_NAME}"
  )
install(DIRECTORY ${suitesparse_BUILD_DIR}/lib/
  DESTINATION "${CMAKE_INSTALL_PREFIX}/lib/"
  )

