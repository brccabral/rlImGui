
add_executable(simple)
target_sources(simple
    PRIVATE simple.cpp
)
target_link_libraries(simple PRIVATE rlImGui)


add_executable(editor)
target_sources(editor
    PRIVATE editor.cpp
)
target_link_libraries(editor PRIVATE rlImGui)


add_executable(imgui_style_example)
target_sources(imgui_style_example
    PRIVATE imgui_style_example.cpp
)
target_link_libraries(imgui_style_example PRIVATE rlImGui)


add_executable(docking_example)
target_sources(docking_example
    PRIVATE docking_example.cpp
)
target_link_libraries(docking_example PRIVATE rlImGui)


if (NOT NO_FONT_AWESOME)
add_executable(asset_browser)
target_sources(asset_browser
    PRIVATE asset_browser/main.cpp asset_browser/asset_browser.cpp asset_browser/imgui_utils.cpp asset_browser/item_views.cpp
        asset_browser/asset_browser.h asset_browser/imgui_utils.h asset_browser/item_view.h
)
target_link_libraries(asset_browser PRIVATE rlImGui)
endif ()
