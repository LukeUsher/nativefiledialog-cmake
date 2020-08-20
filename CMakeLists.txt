include(CheckIncludeFile)
set(SOURCES src/nfd_common.c)

macro(REQUIRE_INCLUDE_FILE path name)
    CHECK_INCLUDE_FILE(${path} ${name})
    if (NOT ${name})
        message(FATAL_ERROR "${path} not found")
    endif ()
endmacro()

# add specific implementations
if (WIN32)
    REQUIRE_INCLUDE_FILE(windows.h HAS_WINDOWS)
    list(APPEND SOURCES src/nfd_win.cpp)
elseif (APPLE)
    REQUIRE_INCLUDE_FILE(AppKit/AppKit.h HAS_APPKIT)
    list(APPEND SOURCES src/nfd_cocoa.m)
elseif (UNIX)
    FIND_PACKAGE(PkgConfig REQUIRED)
    PKG_CHECK_MODULES(GTK3 REQUIRED gtk+-3.0)
    INCLUDE_DIRECTORIES(${GTK3_INCLUDE_DIRS})
    LINK_DIRECTORIES(${GTK_LIBRARY_DIRS})
    list(APPEND SOURCES src/nfd_gtk.c)
elseif (UNIX)
    message(FATAL_ERROR "Cannot detect your system, please report to https://github.com/aarcangeli/nativefiledialog-cmake/issues")
endif ()

add_library(nativefiledialog STATIC ${SOURCES})

target_include_directories(nativefiledialog PUBLIC 
						$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/include>
						"$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>")
if (UNIX AND NOT APPLE)
    target_link_libraries(nativefiledialog PUBLIC ${GTK3_LIBRARIES})
endif ()
