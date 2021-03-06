cmake_minimum_required(VERSION 2.8)

project(vulkan_replay)

include("${SRC_DIR}/build_options.cmake")

execute_process(COMMAND ${PYTHON_EXECUTABLE} ${SCRIPTS_DIR}/vktrace_generate.py AllPlatforms vktrace-replay-vk-funcs     VK_VERSION_1_0 OUTPUT_FILE ${GENERATED_FILES_DIR}/vkreplay_vk_func_ptrs.h)
execute_process(COMMAND ${PYTHON_EXECUTABLE} ${SCRIPTS_DIR}/vktrace_generate.py AllPlatforms vktrace-replay-c            VK_VERSION_1_0 OUTPUT_FILE ${GENERATED_FILES_DIR}/vkreplay_vk_replay_gen.cpp)
execute_process(COMMAND ${PYTHON_EXECUTABLE} ${SCRIPTS_DIR}/vktrace_generate.py AllPlatforms vktrace-replay-obj-mapper-h VK_VERSION_1_0 OUTPUT_FILE ${GENERATED_FILES_DIR}/vkreplay_vk_objmapper.h)

if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
set(OS_REPLAYER_LIBS
    xcb
)
endif()

if (${CMAKE_SYSTEM_NAME} MATCHES "Windows" OR
    ${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
set(OS_REPLAYER_LIBS  )
endif()

set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")

set(SRC_LIST
    ${SRC_LIST}
    vkreplay.cpp
    vkreplay_settings.cpp
    vkreplay_vkreplay.cpp
    vkreplay_vkdisplay.cpp
    ${GENERATED_FILES_DIR}/vkreplay_vk_replay_gen.cpp
)

set (HDR_LIST
    vkreplay.h
    vkreplay_settings.h
    vkreplay_vkdisplay.h
    vkreplay_vkreplay.h
    ${GENERATED_FILES_DIR}/vkreplay_vk_func_ptrs.h
    ${GENERATED_FILES_DIR}/vkreplay_vk_objmapper.h
    ${GENERATED_FILES_DIR}/vktrace_vk_packet_id.h
    ${GENERATED_FILES_DIR}/vktrace_vk_vk_packets.h
)

include_directories(
    codegen
    ${SRC_DIR}/vktrace_replay
    ${SRC_DIR}/vktrace_common
    ${SRC_DIR}/thirdparty
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${GENERATED_FILES_DIR}
    ${VKTRACE_VULKAN_INCLUDE_DIR}
    ${CMAKE_BINARY_DIR}
)
# needed for vktraceviewer_vk library which is shared
if (NOT MSVC)
    add_compiler_flag("-fPIC")
endif()

add_definitions(-DAPI_LOWERCASE="${API_LOWERCASE}")

add_library(${PROJECT_NAME} STATIC ${SRC_LIST} ${HDR_LIST})

if(WIN32)
    add_dependencies(${PROJECT_NAME} generate_helper_files "${API_LOWERCASE}-${MAJOR}")
else()
    add_dependencies(${PROJECT_NAME} generate_helper_files "${API_LOWERCASE}")
endif()

target_link_libraries(${PROJECT_NAME}
    ${OS_REPLAYER_LIBS}
    ${VKTRACE_VULKAN_LIB}
    vktrace_common
)

build_options_finalize()
