cmake_minimum_required(VERSION 3.18...4.2)

project(UETest)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 编译选项（移除-fno-rtti -fno-exceptions避免ImGui报错）
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -w -Wno-error=format-security -fvisibility=hidden -ffunction-sections -fdata-sections")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w -Wno-error=format-security -Wno-error=c++11-narrowing -fvisibility=hidden -Wall -fomit-frame-pointer")

# Release 优化选项
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -g0 -s")
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3 -g0 -s")

# 链接器选项
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--gc-sections,--strip-all")

# 提前导入android native glue头文件
include_directories("native_app_glue")


# 全部头文件目录
include_directories(
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_CURRENT_SOURCE_DIR}/include
        ${CMAKE_CURRENT_SOURCE_DIR}/include/ImGui
        ${CMAKE_CURRENT_SOURCE_DIR}/include/InlineHook
        ${CMAKE_CURRENT_SOURCE_DIR}/include/MainLooper
)

# 添加 native_app_glue 源码（NDK 提供的胶水代码）
add_library(native_app_glue STATIC
        native_app_glue/android_native_app_glue.c
)

# 为其设置头文件搜索路径（这样 Game 也能继承）
target_include_directories(native_app_glue PUBLIC
        native_app_glue/native_app_glue
)

# 收集源文件
file(GLOB_RECURSE IMGUI_SOURCES
        ${CMAKE_CURRENT_SOURCE_DIR}/include/ImGui/*.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/include/ImGui/*.c
)

file(GLOB_RECURSE INLINEHOOK_SOURCES
        ${CMAKE_CURRENT_SOURCE_DIR}/include/InlineHook/*.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/include/InlineHook/*.c
)

file(GLOB_RECURSE MAINLOOPER_SOURCES
        ${CMAKE_CURRENT_SOURCE_DIR}/include/MainLooper/*.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/include/MainLooper/*.c
)

file(GLOB ROOT_SOURCES
        ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/*.c
)

# 创建核心动态库 Game（安卓so输出）
add_library(
        Game
        SHARED
        ${ROOT_SOURCES}
        ${IMGUI_SOURCES}
        ${INLINEHOOK_SOURCES}
        ${MAINLOOPER_SOURCES}
)

# 正确写法：第一个参数是目标Game，后面跟所有需要链接的库
target_link_libraries(Game
        native_app_glue
        android
        log
        EGL
        GLESv1_CM
        GLESv2
        GLESv3
        z
        m
        dl
)
