cmake_minimum_required(VERSION 2.8)
project(vkreplay)

include("${SRC_DIR}/build_options.cmake")

set(SRC_LIST
    ${SRC_LIST}
    vkreplay_factory.h
    vkreplay_seq.h
    vkreplay_window.h
    vkreplay_main.cpp
    vkreplay_seq.cpp
    vkreplay_factory.cpp
    ../../../layersvt/screenshot_parsing.h
    ../../../layersvt/screenshot_parsing.cpp
)

include_directories(
    ${SRC_DIR}/vktrace_replay
    ${SRC_DIR}/vktrace_common
    ${SRC_DIR}/thirdparty
    ${CMAKE_CURRENT_SOURCE_DIR}/../vktrace_extensions/vktracevulkan/vkreplay/
    ${VULKAN_TOOLS_SOURCE_DIR}/layersvt
)

set (LIBRARIES vktrace_common vulkan_replay)

add_executable(${PROJECT_NAME} ${SRC_LIST})

if(WIN32)
    add_dependencies(${PROJECT_NAME} "${API_LOWERCASE}-${MAJOR}")
else()
    add_dependencies(${PROJECT_NAME} "${API_LOWERCASE}")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif()

target_link_libraries(${PROJECT_NAME}
    ${LIBRARIES}
)

build_options_finalize()
