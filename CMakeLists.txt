cmake_minimum_required(VERSION 3.22.0)

project(imgui LANGUAGES CXX VERSION 2.0)

option(BUILD_SHARED_LIBS "Build shared libraries" OFF)

find_package(SDL3 REQUIRED)
find_package(VulkanHeaders REQUIRED)
find_package(VulkanLoader REQUIRED)

set(TARGET_NAME imgui)

set(IMGUI_SOURCES imgui/imgui.cpp
        imgui/imgui_draw.cpp
        imgui/imgui_tables.cpp
        imgui/imgui_widgets.cpp
        imgui/imgui_demo.cpp
        imgui/backends/imgui_impl_vulkan.cpp
        imgui/backends/imgui_impl_sdl3.cpp)

set(IMGUI_HEADERS imgui/imgui.h
        imgui/imgui_internal.h
        imgui/imconfig.h
        imgui/imstb_rectpack.h
        imgui/imstb_textedit.h
        imgui/imstb_truetype.h
        imgui/backends/imgui_impl_vulkan.h
        imgui/backends/imgui_impl_sdl3.h)

if(WIN32)
    set(IMGUI_SOURCES ${IMGUI_SOURCES} imgui/backends/imgui_impl_dx12.cpp)
    set(IMGUI_HEADERS ${IMGUI_HEADERS} imgui/backends/imgui_impl_dx12.h)
endif()


set(IMGUI_BUILD_TYPE STATIC)
if(BUILD_SHARED_LIBS)
    set(IMGUI_BUILD_TYPE SHARED)
endif()

add_library(${TARGET_NAME} ${IMGUI_BUILD_TYPE} ${IMGUI_SOURCES} ${IMGUI_HEADERS})
target_link_libraries(${TARGET_NAME} PRIVATE Vulkan::Headers Vulkan::Loader SDL3::SDL3-shared)

target_include_directories(${TARGET_NAME} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/imgui> 
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/imgui/backends>
        $<INSTALL_INTERFACE:include/imgui>)

if(WIN32)
    target_link_libraries(${TARGET_NAME} PRIVATE dxgi dxguid)
    if(BUILD_SHARED_LIBS)
        target_compile_definitions(${TARGET_NAME} PRIVATE IMGUI_API=__declspec\(dllexport\))
        target_compile_definitions(${TARGET_NAME} INTERFACE IMGUI_API=__declspec\(dllimport\))
    endif()
endif()

set(IMGUI_GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(IMGUI_VERSION_CONFIG "${IMGUI_GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(IMGUI_PROJECT_CONFIG "${IMGUI_GENERATED_DIR}/${PROJECT_NAME}Config.cmake")
set(IMGUI_TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(IMGUI_CONFIG_INSTALL_DIR "lib/cmake/${PROJECT_NAME}")
set(IMGUI_NAMESPACE "imgui::")
set(IMGUI_VERSION ${PROJECT_VERSION})

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${IMGUI_VERSION_CONFIG}" VERSION ${IMGUI_VERSION} COMPATIBILITY SameMajorVersion
)
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/config.cmake.in" "${IMGUI_PROJECT_CONFIG}" @ONLY)

# Install cmake config files
install(
  FILES "${IMGUI_PROJECT_CONFIG}" "${IMGUI_VERSION_CONFIG}"
  DESTINATION "${IMGUI_CONFIG_INSTALL_DIR}")

install(
  EXPORT "${IMGUI_TARGETS_EXPORT_NAME}"
  NAMESPACE "${IMGUI_NAMESPACE}"
  DESTINATION "${IMGUI_CONFIG_INSTALL_DIR}")

install(TARGETS imgui
        EXPORT ${IMGUI_TARGETS_EXPORT_NAME} 
		RUNTIME DESTINATION "bin"
        	ARCHIVE DESTINATION "lib"
        	LIBRARY DESTINATION "lib")
install(FILES ${IMGUI_HEADERS}
        DESTINATION "include/imgui")
