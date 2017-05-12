CMAKE_MINIMUM_REQUIRED(VERSION 3.2)

# Project definition

PROJECT(LibXDiff)

# Package name and version

SET(PACKAGE_NAME xdiff)

SET(PACKAGE_MAJOR_VERSION 0)
SET(PACKAGE_VERSION "${PACKAGE_MAJOR_VERSION}.23")

# Some options to customise the build

OPTION(ENABLE_TESTS "Enable the tests to be built" OFF)
OPTION(ENABLE_TOOLS "Enable the tools to be built" OFF)

# Check for some include files and functions

INCLUDE(CheckIncludeFiles)
INCLUDE(CheckFunctionExists)

CHECK_INCLUDE_FILES(limits.h HAVE_LIMITS_H)
CHECK_FUNCTION_EXISTS(memchr HAVE_MEMCHR)
CHECK_FUNCTION_EXISTS(memcmp HAVE_MEMCMP)
CHECK_FUNCTION_EXISTS(memcpy HAVE_MEMCPY)
CHECK_FUNCTION_EXISTS(memset HAVE_MEMSET)
CHECK_INCLUDE_FILES(stdio.h HAVE_STDIO_H)
CHECK_INCLUDE_FILES(stdlib.h HAVE_STDLIB_H)
CHECK_INCLUDE_FILES(string.h HAVE_STRING_H)
CHECK_FUNCTION_EXISTS(strlen HAVE_STRLEN)
CHECK_INCLUDE_FILES(unistd.h HAVE_UNISTD_H)

# Configure LibXDiff

CONFIGURE_FILE(config.h.in config.h)

ADD_DEFINITIONS(-DHAVE_CONFIG_H)

INCLUDE_DIRECTORIES(${CMAKE_BINARY_DIR})

# Build LibXDiff

IF(WIN32)
    ADD_DEFINITIONS(-DLIBXDIFF_DLL_EXPORT)
ENDIF()

ADD_LIBRARY(${PACKAGE_NAME} SHARED
    xdiff/xadler32.c
    xdiff/xalloc.c
    xdiff/xbdiff.c
    xdiff/xbpatchi.c
    xdiff/xdiffi.c
    xdiff/xemit.c
    xdiff/xmerge3.c
    xdiff/xmissing.c
    xdiff/xpatchi.c
    xdiff/xprepare.c
    xdiff/xrabdiff.c
    xdiff/xrabply.c
    xdiff/xutils.c
    xdiff/xversion.c
)

SET_TARGET_PROPERTIES(${PACKAGE_NAME} PROPERTIES VERSION ${PACKAGE_VERSION})
SET_TARGET_PROPERTIES(${PACKAGE_NAME} PROPERTIES SOVERSION ${PACKAGE_MAJOR_VERSION})

# Install LibXDiff

INSTALL(TARGETS ${PACKAGE_NAME}
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

INSTALL(FILES xdiff/xdiff.h
        DESTINATION include)

# Set the RPATH information on Linux and macOS

IF(ENABLE_TESTS OR ENABLE_TOOLS)
    IF(APPLE)
        SET(CMAKE_INSTALL_RPATH "@executable_path/../lib")
    ELSEIF(NOT WIN32)
        SET(CMAKE_INSTALL_RPATH "$ORIGIN/../lib")
    ENDIF()
ENDIF()

# Build the tests, if requested

IF(ENABLE_TESTS)
    INCLUDE_DIRECTORIES(xdiff)

    ADD_EXECUTABLE(xdiff_test
        test/xdiff_test.c
        test/xtestutils.c
    )

    TARGET_LINK_LIBRARIES(xdiff_test ${PACKAGE_NAME})

    ADD_EXECUTABLE(xregression
        test/xregression.c
        test/xtestutils.c
    )

    TARGET_LINK_LIBRARIES(xregression ${PACKAGE_NAME})

    INSTALL(TARGETS xregression
            RUNTIME DESTINATION bin)
ENDIF()

# Build the tools, if requested

IF(ENABLE_TOOLS)
    ADD_EXECUTABLE(xrabin
        tools/xrabin.c
    )

    INSTALL(TARGETS xrabin
            RUNTIME DESTINATION bin)
ENDIF()
