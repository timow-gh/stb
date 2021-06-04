cmake_minimum_required(VERSION 3.19)
project(stb
        VERSION 2.36
        LANGUAGES C)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake)

option(${PROJECT_NAME}_INSTALL_LIB "Enable install rule" ON)

if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type selected, default to Release")
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
endif ()

if (${PROJECT_NAME}_INSTALL_LIB)
    include(GNUInstallDirs)
    if (NOT CMAKE_GENERATOR STREQUAL "Xcode")
        file(RELATIVE_PATH relDir
             ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}
             ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
        set(CMAKE_INSTALL_RPATH $ORIGIN $ORIGIN/${relDir})
    endif ()
endif ()

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

set(stb_HEADERS
    stb.h
    stb_c_lexer.h
    stb_connected_components.h
    stb_divide.h
    stb_ds.h
    stb_dxt.h
    stb_easy_font.h
    stb_herringbone_wang_tile.h
    stb_hexwave.h
    stb_image.h
    stb_image_resize.h
    stb_image_write.h
    stb_include.h
    stb_leakcheck.h
    stb_perlin.h
    stb_rect_pack.h
    stb_sprintf.h
    stb_textedit.h
    stb_tilemap_editor.h
    stb_truetype.h
    stb_vorbis.c
    stb_voxel_render.h)

add_library(stb INTERFACE)
target_include_directories(stb INTERFACE $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/>)

set(TARGET_ALIAS Stb)
add_library(${PROJECT_NAME}::${TARGET_ALIAS} ALIAS stb)
set_target_properties(stb PROPERTIES EXPORT_NAME ${TARGET_ALIAS})

if (${PROJECT_NAME}_INSTALL_LIB)
    install(FILES ${stb_HEADERS} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/${TARGET_ALIAS}")

    install(TARGETS stb
            EXPORT StbTargetsExportSet
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
            INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

    include(CMakePackageConfigHelpers)

    set(${PROJECT_NAME}_INSTALL_CMAKEDIR
        "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
        CACHE STRING "Path to install ${PROJECT_NAME} Config*.cmake files to.")
    set(${PROJECT_NAME}_MODULE_INSTALL_DIR
        "${CMAKE_INSTALL_LIBDIR}/cmake"
        CACHE STRING "Path to install ${PROJECT_NAME}'s .cmake  module files to.")

    install(EXPORT StbTargetsExportSet
            NAMESPACE Stb::
            FILE StbTargets.cmake
            DESTINATION ${${PROJECT_NAME}_INSTALL_CMAKEDIR})

    write_basic_package_version_file(
            ${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
            VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}
            COMPATIBILITY SameMajorVersion)

    configure_package_config_file(
            ${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in
            ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
            INSTALL_DESTINATION ${${PROJECT_NAME}_INSTALL_CMAKEDIR}
            PATH_VARS ${PROJECT_NAME}_MODULE_INSTALL_DIR
            NO_SET_AND_CHECK_MACRO
            NO_CHECK_REQUIRED_COMPONENTS_MACRO)

    install(FILES
            "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
            "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
            DESTINATION ${${PROJECT_NAME}_INSTALL_CMAKEDIR})
endif ()
