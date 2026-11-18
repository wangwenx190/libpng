cmake_minimum_required(VERSION 3.25)
project(libpng VERSION 1.8.0 LANGUAGES C)
include(GNUInstallDirs)
find_package(ZLIB REQUIRED)

option(PNG_STATIC "Build the static library" ON)
option(PNG_SHARED "Build the shared library" ON)
configure_file(pnglibconf.h.prebuilt pnglibconf.h COPYONLY)
file(GLOB png_sources CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/png*.c")

function(add_png target type)
    add_library(${target} ${type} ${png_sources})
    target_include_directories(${target} PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>"
        "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>")
    target_link_libraries(${target} PUBLIC ZLIB::ZLIB)
    if(NOT WIN32)
        target_link_libraries(${target} PRIVATE m)
    endif()
    set_target_properties(${target} PROPERTIES POSITION_INDEPENDENT_CODE ON)
    if(NOT SKIP_INSTALL_ALL)
        install(TARGETS ${target})
    endif()
endfunction()
if(PNG_STATIC)
    add_png(png_static STATIC)
endif()
if(PNG_SHARED)
    add_png(png_shared SHARED)
    set_target_properties(png_shared PROPERTIES OUTPUT_NAME png VERSION ${PROJECT_VERSION}
        SOVERSION 18 WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()
if(NOT SKIP_INSTALL_ALL)
    install(FILES png.h pngconf.h "${CMAKE_CURRENT_BINARY_DIR}/pnglibconf.h"
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")
    install(FILES LICENSE.md DESTINATION "${CMAKE_INSTALL_DATADIR}/licenses/libpng")
endif()

if(WIN32 AND TARGET png_shared)
    enable_language(RC)
    configure_file(cmake/LibraryVersion.rc.in LibraryVersion.rc @ONLY)
    target_sources(png_shared PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/LibraryVersion.rc")
endif()
