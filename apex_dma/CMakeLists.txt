cmake_minimum_required(VERSION 3.5)

project(apex_dma VERSION 0.1.0 LANGUAGES C CXX)

include(CTest)
enable_testing()

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)

set(CMAKE_CXX_STANDARD 17)
set(CXX_STANDARD_REQUIRED ON)

set(CMAKE_CXX_FLAGS "-g -Wall -Wformat -march=native")
set(CMAKE_BUILD_TYPE "Release")

# memflow
include(ExternalProject)
ExternalProject_Add(
    memflow_win32_ffi
    DOWNLOAD_COMMAND ""
    CONFIGURE_COMMAND ""
    BINARY_DIR ${PROJECT_SOURCE_DIR}/memflow/memflow-win32-ffi/
    BUILD_COMMAND cargo build --release
    INSTALL_COMMAND ""
    LOG_BUILD ON)
ExternalProject_Add(
    memflow_qemu_procfs
    DOWNLOAD_COMMAND ""
    CONFIGURE_COMMAND ""
    BINARY_DIR ${PROJECT_SOURCE_DIR}/memflow-qemu/
    BUILD_COMMAND cargo build --release --all-features
    INSTALL_COMMAND cp ${PROJECT_SOURCE_DIR}/memflow-qemu/target/release/libmemflow_qemu_procfs.so ${CMAKE_BINARY_DIR}/
    LOG_BUILD ON)
find_package(Threads REQUIRED)

# GLFW
set(GLFW_DIR ${PROJECT_SOURCE_DIR}/Client/glfw) # Set this to point to an up-to-date GLFW repo
option(GLFW_BUILD_EXAMPLES "Build the GLFW example programs" OFF)
option(GLFW_BUILD_TESTS "Build the GLFW test programs" OFF)
option(GLFW_BUILD_DOCS "Build the GLFW documentation" OFF)
option(GLFW_INSTALL "Generate installation target" OFF)
option(GLFW_DOCUMENT_INTERNALS "Include internals in documentation" OFF)
add_subdirectory(${GLFW_DIR} binary_dir EXCLUDE_FROM_ALL)
include_directories(${GLFW_DIR}/include)

# Dear ImGui
set(IMGUI_DIR ${PROJECT_SOURCE_DIR}/Client/imgui)
include_directories(${IMGUI_DIR} ${IMGUI_DIR}/backends)


# apex_dma
add_executable(apex_dma
    apex_dma.cpp Game.cpp Math.cpp memory.cpp
    ${PROJECT_SOURCE_DIR}/Client/main.cpp
    ${PROJECT_SOURCE_DIR}/Client/overlay.cpp
    ${IMGUI_DIR}/imgui.cpp
    ${IMGUI_DIR}/imgui_demo.cpp
    ${IMGUI_DIR}/imgui_draw.cpp
    ${IMGUI_DIR}/imgui_tables.cpp
    ${IMGUI_DIR}/imgui_widgets.cpp
    ${IMGUI_DIR}/backends/imgui_impl_glfw.cpp
    ${IMGUI_DIR}/backends/imgui_impl_opengl3.cpp)

add_dependencies(apex_dma memflow_win32_ffi memflow_qemu_procfs)
target_link_libraries(
    apex_dma
    "${PROJECT_SOURCE_DIR}/memflow/target/release/libmemflow_win32_ffi.a"
    glfw
    GL
    ${CMAKE_THREAD_LIBS_INIT}
    ${CMAKE_DL_LIBS}
    m)
target_include_directories(
    apex_dma PUBLIC
    "${PROJECT_SOURCE_DIR}/memflow/memflow-win32-ffi"
    "${PROJECT_SOURCE_DIR}/memflow/memflow-ffi")