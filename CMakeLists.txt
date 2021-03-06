cmake_minimum_required(VERSION 2.6)

project(UriParser C)

if("${CMAKE_BUILD_TYPE}" STREQUAL "")
  set(CMAKE_BUILD_TYPE Release)
endif()

set(INSTALL_BIN_DIR "${CMAKE_INSTALL_PREFIX}/bin" CACHE PATH "Installation directory for executables")
set(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
set(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "Installation directory for headers")
set(INSTALL_MAN_DIR "${CMAKE_INSTALL_PREFIX}/share/man" CACHE PATH "Installation directory for manual pages")
set(INSTALL_PKGCONFIG_DIR "${CMAKE_INSTALL_PREFIX}/share/pkgconfig" CACHE PATH "Installation directory for pkgconfig (.pc) files")

set(VERSION_MAJOR "0")
set(VERSION_MINOR "7")
set(VERSION_PATCH "8")
set(VERSION_STRING "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

option(BUILD_SHARED_LIBS  "Build shared libs." ON)
option(BUILD_TESTS "Build demos." ON)
option(BUILD_DEMOS  "Build demos." ON)

set(INSTALL_BIN_DIR "${CMAKE_INSTALL_PREFIX}/bin" CACHE PATH "Installation directory for executables")
set(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
set(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "Installation directory for headers")
set(INSTALL_MAN_DIR "${CMAKE_INSTALL_PREFIX}/share/man" CACHE PATH "Installation directory for manual pages")
set(INSTALL_PKGCONFIG_DIR "${CMAKE_INSTALL_PREFIX}/share/pkgconfig" CACHE PATH "Installation directory for pkgconfig (.pc) files")

if(WIN32 AND NOT CYGWIN)
  set(DEF_INSTALL_CMAKE_DIR CMake)
else()
  set(DEF_INSTALL_CMAKE_DIR lib/CMake/UriParser)
endif()
set(INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH
  "Installation directory for CMake files")

include (CheckIncludeFile)
include (CheckFunctionExists)

check_include_file("wchar.h" HAVE_WCHAR_H)
check_include_file("stdio.h" HAVE_STDIO_H)

check_function_exists(wprintf HAVE_WPRINTF)

set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

option(ENABLE_SIZEDOWN  "enable size down: -DURI_SIZEDOWN" ON)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DURI_SIZEDOWN")

option(DISABLE_CHAR "disable char: -DURI_NO_ANSI" OFF)
if(DISABLE_CHAR)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DURI_NO_ANSI")
endif()

option(DISABLE_WCHAR_T "disable wchar_t: -DURI_NO_UNICODE " OFF)
if(DISABLE_WCHAR_T)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DURI_NO_UNICODE")
endif()

if(DISABLE_WCHAR_T AND DISABLE_CHAR)
  #both can't be ON
endif()

if(DISABLE_WCHAR_T OR DISABLE_CHAR)
  #disable testing
endif()

set(URIPARSER_SRCS
  src/UriCommon.c
  src/UriCommon.h
  src/UriCompare.c
  src/UriEscape.c
  src/UriFile.c
  src/UriIp4.c
  src/UriIp4Base.c
  src/UriIp4Base.h
  src/UriNormalize.c
  src/UriNormalizeBase.c
  src/UriNormalizeBase.h
  src/UriParse.c
  src/UriParseBase.c
  src/UriParseBase.h
  src/UriQuery.c
  src/UriRecompose.c
  src/UriResolve.c
  src/UriShorten.c
  )

set(URIPARSER_INCS
  include/uriparser/Uri.h
  include/uriparser/UriBase.h
  include/uriparser/UriDefsAnsi.h
  include/uriparser/UriDefsConfig.h
  include/uriparser/UriDefsUnicode.h
  include/uriparser/UriIp4.h
  )
include_directories(${CMAKE_SOURCE_DIR}/include)
set(LIB_NAME uriparser)

set(CONFIG_TYPE "NOCONFIG")
if(NOT "${CMAKE_BUILD_TYPE}" STREQUAL "")
  string(TOUPPER ${CMAKE_BUILD_TYPE} CONFIG_TYPE)
endif()

add_library(uriparser ${URIPARSER_SRCS})
set_target_properties(uriparser PROPERTIES SOVERSION ${VERSION_STRING})

if(NOT WIN32)
  configure_file( ${CMAKE_CURRENT_SOURCE_DIR}/cmake/liburiparser.pc.cmakein
    ${CMAKE_CURRENT_BINARY_DIR}/liburiparser.pc @ONLY)
  
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/liburiparser.pc
    DESTINATION
    "${INSTALL_PKGCONFIG_DIR}")
endif()
export(TARGETS uriparser
  FILE "${CMAKE_BINARY_DIR}/UriParserTargets.cmake")

install(TARGETS uriparser
  EXPORT UriParserTargets
  RUNTIME DESTINATION "${INSTALL_BIN_DIR}"
  ARCHIVE DESTINATION "${INSTALL_LIB_DIR}"
  LIBRARY DESTINATION "${INSTALL_LIB_DIR}" )

install(FILES ${URIPARSER_INCS} DESTINATION "${INSTALL_INC_DIR}")
 
if(BUILD_TESTS)
  enable_testing()
endif()

set(URIPARSER_VERSION ${VERSION_STRING}) 

# Make relative paths absolute (needed later on)
foreach(p LIB BIN INCLUDE CMAKE)
  set(var INSTALL_${p}_DIR)
  if(NOT IS_ABSOLUTE "${${var}}")
    set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
  endif()
endforeach()

file(RELATIVE_PATH REL_INCLUDE_DIR "${INSTALL_CMAKE_DIR}"
  "${INSTALL_INC_DIR}")
# ... for the build tree
set(CONF_INCLUDE_DIRS "${CMAKE_SOURCE_DIR}/include")
configure_file(cmake/UriParserConfig.cmake.in
  "${PROJECT_BINARY_DIR}/UriParserConfig.cmake" @ONLY)

# ... for the install tree
set(CONF_INCLUDE_DIRS "\${INSTALL_CMAKE_DIR}/${REL_INCLUDE_DIR}")
configure_file(cmake/UriParserConfig.cmake.in
  "${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/UriParserConfig.cmake" @ONLY)
# ... for both
configure_file(cmake/UriParserConfigVersion.cmake.in
  "${CMAKE_BINARY_DIR}/UriParserConfigVersion.cmake" @ONLY)
 
install(FILES
  "${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/UriParserConfig.cmake"
  "${CMAKE_BINARY_DIR}/UriParserConfigVersion.cmake"
  DESTINATION ${DEF_INSTALL_CMAKE_DIR} )

# Install the export set for use with the install-tree
install(EXPORT UriParserTargets DESTINATION ${DEF_INSTALL_CMAKE_DIR})

