cmake_minimum_required(VERSION 3.15)
project(example_glfw_opengl3 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

find_package(glfw3 3.3 QUIET)

if(NOT glfw3_FOUND)
    message(STATUS "System GLFW not found, using FetchContent")
    include(FetchContent)
    
    FetchContent_Declare(
        glfw
        GIT_REPOSITORY https://github.com/glfw/glfw.git
        GIT_TAG 3.3.8
        GIT_SHALLOW TRUE
    )
    
    set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
    set(GLFW_INSTALL OFF CACHE BOOL "" FORCE)
    
    FetchContent_MakeAvailable(glfw)
    
    set(GLFW_LIB glfw)
else()
    message(STATUS "Using system GLFW")
    set(GLFW_LIB glfw)
endif()

set(IMGUI_VERSION "1.92.1" CACHE STRING "ImGui version")
include(FetchContent)
FetchContent_Declare(
    imgui
    GIT_REPOSITORY https://github.com/ocornut/imgui.git
    GIT_TAG v${IMGUI_VERSION}
    GIT_SHALLOW TRUE
)
FetchContent_MakeAvailable(imgui)

set(SOURCES
    src/main.cpp
    ${imgui_SOURCE_DIR}/imgui.cpp
    ${imgui_SOURCE_DIR}/imgui_demo.cpp
    ${imgui_SOURCE_DIR}/imgui_draw.cpp
    ${imgui_SOURCE_DIR}/imgui_tables.cpp
    ${imgui_SOURCE_DIR}/imgui_widgets.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp
    ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl3.cpp
)

add_executable(${PROJECT_NAME} ${SOURCES})

target_include_directories(${PROJECT_NAME} PRIVATE
    ${imgui_SOURCE_DIR}
    ${imgui_SOURCE_DIR}/backends
)

target_compile_options(${PROJECT_NAME} PRIVATE
    $<$<CXX_COMPILER_ID:GNU,Clang>:-Wall -Wformat>
)

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    find_package(OpenGL REQUIRED)
    target_link_libraries(${PROJECT_NAME} PRIVATE OpenGL::GL)
    
elseif(APPLE)
    find_library(COCOA_LIB Cocoa)
    find_library(IOKIT_LIB IOKit)
    find_library(COREVIDEO_LIB CoreVideo)
    
    target_compile_definitions(${PROJECT_NAME} PRIVATE GL_SILENCE_DEPRECATION)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        ${COCOA_LIB}
        ${IOKIT_LIB}
        ${COREVIDEO_LIB}
        "-framework OpenGL"
    )

elseif(WIN32)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        opengl32
        gdi32
        imm32
    )
    target_compile_definitions(${PROJECT_NAME} PRIVATE _CRT_SECURE_NO_WARNINGS)
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE ${GLFW_LIB})

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)