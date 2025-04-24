cmake_minimum_required(VERSION 3.23)
project(rlImGui VERSION 0.1.0)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(RLIMGUI_BUILD_EXAMPLES "Build examples" OFF)
option(BUILD_LIBTYPE_SHARED "Build type, shared OFF by default" OFF)
option(NO_FONT_AWESOME "Do not use Font Awesome" OFF)

include(FetchContent)
include(GNUInstallDirs)

if (BUILD_LIBTYPE_SHARED)
    add_library(rlImGui SHARED)
    target_compile_definitions(rlImGui
                           INTERFACE $<BUILD_INTERFACE:BUILD_LIBTYPE_SHARED>
                           $<INSTALL_INTERFACE:USE_LIBTYPE_SHARED>
    )
    if (WIN32)
        target_compile_definitions(rlImGui INTERFACE "IMGUI_API=__declspec(dllimport)")
    endif ()
else ()
    add_library(rlImGui STATIC)
endif ()
add_library(rlImGui::rlImGui ALIAS rlImGui)
set_target_properties(rlImGui PROPERTIES EXPORT_NAME "rlImGui")

target_sources(rlImGui
    PUBLIC
        FILE_SET rlImGui_headers
        TYPE HEADERS
        FILES rlImGui.h rlImGuiColors.h imgui_impl_raylib.h
    PRIVATE
        rlImGui.cpp
)

#find_package(imgui QUIET)
if (NOT imgui_FOUND)
    message(STATUS "Fetching imgui")
    FetchContent_Declare(
        imgui
        URL      https://github.com/ocornut/imgui/archive/refs/tags/v1.91.9b.zip
        URL_HASH MD5=913db10a6ef52623b118142aafba7438
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
    )
    FetchContent_MakeAvailable(imgui)
    add_library(imgui STATIC)
    add_library(imgui::imgui ALIAS imgui)
    target_sources(imgui
        PUBLIC
            FILE_SET HEADERS
                FILES
                ${imgui_SOURCE_DIR}/imgui.h
                ${imgui_SOURCE_DIR}/imconfig.h
                ${imgui_SOURCE_DIR}/imgui_internal.h
                ${imgui_SOURCE_DIR}/imstb_rectpack.h
                ${imgui_SOURCE_DIR}/imstb_textedit.h
                ${imgui_SOURCE_DIR}/imstb_truetype.h
                BASE_DIRS ${imgui_SOURCE_DIR}
        PRIVATE
            ${imgui_SOURCE_DIR}/imgui.cpp
            ${imgui_SOURCE_DIR}/imgui_demo.cpp
            ${imgui_SOURCE_DIR}/imgui_draw.cpp
            ${imgui_SOURCE_DIR}/imgui_tables.cpp
            ${imgui_SOURCE_DIR}/imgui_widgets.cpp
    )
    target_include_directories(rlImGui PUBLIC $<BUILD_INTERFACE:${imgui_SOURCE_DIR}>)
    if (BUILD_LIBTYPE_SHARED)
        set_property(TARGET imgui PROPERTY POSITION_INDEPENDENT_CODE ON)
        if (MSVC)
            target_link_options(rlImGui PRIVATE "/WHOLEARCHIVE:$<TARGET_FILE:imgui>")
        else()
            target_link_libraries(rlImGui PRIVATE -Wl,--whole-archive imgui -Wl,--no-whole-archive)
        endif()
    endif ()

    file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake.in"
        "@PACKAGE_INIT@\ninclude(\"\${CMAKE_CURRENT_LIST_DIR}/imguiTargets.cmake\")\n"
    )
    include(CMakePackageConfigHelpers)
    write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfigVersion.cmake"
        VERSION 1.91.9
        COMPATIBILITY SameMajorVersion
    )
    configure_package_config_file(
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake"
        INSTALL_DESTINATION lib/cmake/imgui
    )

    install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/imguiConfigVersion.cmake"
        DESTINATION lib/cmake/imgui
    )

    set_target_properties(imgui PROPERTIES EXPORT_NAME "imgui")
    install(TARGETS imgui
        EXPORT imguiTargets
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        INCLUDES DESTINATION include
        FILE_SET HEADERS DESTINATION include
    )
    install(EXPORT imguiTargets
            NAMESPACE imgui::
            DESTINATION lib/cmake/imgui
    )
endif ()
target_link_libraries(rlImGui PRIVATE imgui::imgui)

#find_package(raylib QUIET)
if (NOT raylib_FOUND)
    message(STATUS "Fetching raylib")
    FetchContent_Declare(
        raylib
        URL      https://github.com/raysan5/raylib/archive/refs/tags/5.5.zip
        URL_HASH MD5=fa441b96a43796cd4b0ee2c7150e1d56
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
    )
    FetchContent_MakeAvailable(raylib)
    target_include_directories(rlImGui PUBLIC $<BUILD_INTERFACE:${raylib_SOURCE_DIR}/src>)
    if (BUILD_LIBTYPE_SHARED)
        set_property(TARGET raylib PROPERTY POSITION_INDEPENDENT_CODE ON)
        if (MSVC)
            target_link_options(rlImGui PRIVATE "/WHOLEARCHIVE:$<TARGET_FILE:raylib>")
        else()
            target_link_libraries(rlImGui PRIVATE -Wl,--whole-archive raylib -Wl,--no-whole-archive)
        endif()
    endif ()

    install(TARGETS raylib glfw
        EXPORT raylibTargets
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        INCLUDES DESTINATION include
    )
    install(EXPORT raylibTargets
            DESTINATION lib/cmake/raylib
    )
endif ()
target_link_libraries(rlImGui PRIVATE raylib)


if (NO_FONT_AWESOME)
    target_compile_definitions(rlImGui PUBLIC "DNO_FONT_AWESOME")
    install(TARGETS rlImGui
            EXPORT rlImGuiTargets
            ARCHIVE DESTINATION lib
            FILE_SET rlImGui_headers DESTINATION include
    )
else ()
    target_sources(rlImGui
        PUBLIC
            FILE_SET rlImGui_config
               TYPE HEADERS
               FILES extras/IconsFontAwesome6.h extras/FA6FreeSolidFontData.h
    )
    install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/extras/FontAwsome_LICENSE.txt" DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/extras/FontAwsome_LICENSE.txt")
    install(TARGETS rlImGui
            EXPORT rlImGuiTargets
            ARCHIVE DESTINATION lib
            FILE_SET rlImGui_headers DESTINATION include
            FILE_SET rlImGui_config DESTINATION include
    )
endif ()

install(EXPORT rlImGuiTargets
        NAMESPACE rlImGui::
        DESTINATION lib/cmake/rlImGui
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/rlImGuiConfigVersion.cmake"
    COMPATIBILITY SameMajorVersion
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/rlImGuiConfigVersion.cmake"
    DESTINATION lib/cmake/rlImGui
)

configure_file(cmake/rlImGuiConfig.cmake.in rlImGuiConfig.cmake @ONLY)
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/rlImGuiConfig.cmake"
    DESTINATION lib/cmake/rlImGui
)


if (RLIMGUI_BUILD_EXAMPLES)
    add_subdirectory(examples)
endif ()

