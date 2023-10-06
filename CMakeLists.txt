cmake_minimum_required(VERSION 3.24)

set(CMAKE_BINARY_DIR ${CMAKE_SOURCE_DIR}/build)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}) 


list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include(GetGitVersion)

get_git_version(GIT_VERSION SEM_VER)

include(FetchContent)

FetchContent_Declare(
    freetype
    GIT_REPOSITORY https://github.com/freetype/freetype.git
    OVERRIDE_FIND_PACKAGE
    GIT_TAG        VER-2-13-0    
)
FetchContent_MakeAvailable(freetype)
include_directories(
${freetype_SOURCE_DIR}/src
${freetype_SOURCE_DIR}/include
${freetype_SOURCE_DIR}/include/freetype
)

project(ImPlay VERSION "${SEM_VER}")
set(CMAKE_CXX_STANDARD 20)
add_definitions(-DNOMINMAX)
include(CMakeDependentOption)

option(USE_OPENGL_ES3 "Compile with OpenGL ES 3.0 loader" OFF)
option(USE_PATCHED_GLFW "Use patched GLFW to support additional features" OFF)

cmake_dependent_option(USE_XDG_PORTAL "Use xdg-desktop-portal for file dialogs on Linux" OFF "UNIX;NOT APPLE" OFF)

find_package(Threads REQUIRED)


  add_subdirectory(third_party/glfw)
  set(GLFW_LIBRARIES glfw)

set(LIBROMFS_PROJECT_NAME ${PROJECT_NAME})
set(LIBROMFS_RESOURCE_LOCATION "${CMAKE_SOURCE_DIR}/resources/romfs")
set(OPENGL_LIBRARIES "glad")

add_subdirectory(third_party/glad)
add_subdirectory(third_party/fmt)
add_subdirectory(third_party/natsort)
add_subdirectory(third_party/json)
add_subdirectory(third_party/inipp)
add_subdirectory(third_party/imgui)
add_subdirectory(third_party/nativefiledialog)
add_subdirectory(third_party/libromfs)

set(SOURCE_FILES  
  source/helpers/imgui.cpp
  source/helpers/lang.cpp
  source/helpers/nfd.cpp
  source/helpers/utils.cpp
  source/views/view.cpp
  source/views/command_palette.cpp
  source/views/context_menu.cpp
  source/views/debug.cpp
  source/views/about.cpp
  source/views/quickview.cpp
  source/views/settings.cpp
  source/theme.cpp
  source/config.cpp
  source/mpv.cpp
  source/player.cpp
  source/window.cpp
  source/main.cpp
)
set(MPV_LIBRARY_DIRS "third_party/mpv")
set(MPV_INCLUDE_DIRS "third_party/mpv/include")
set(MPV_LIBRARIES "${CMAKE_SOURCE_DIR}/third_party/mpv/libmpv.dll.a")
set(MPV_DLL "${CMAKE_SOURCE_DIR}/third_party/mpv/libmpv-2.dll")

set(INCLUDE_DIRS include ${MPV_INCLUDE_DIRS} ${GLFW_INCLUDE_DIRS} ${OPENGL_INCLUDE_DIR})
set(LINK_LIBS glad fmt natsort json inipp nfd imgui ${CMAKE_THREAD_LIBS_INIT} ${MPV_LIBRARIES} ${GLFW_LIBRARIES} ${LIBROMFS_LIBRARY})

if(WIN32)
  configure_file(${PROJECT_SOURCE_DIR}/resources/win32/app.rc.in ${PROJECT_BINARY_DIR}/app.rc @ONLY)
  list(APPEND SOURCE_FILES ${PROJECT_BINARY_DIR}/app.rc)
endif()


add_executable(${PROJECT_NAME} ${SOURCE_FILES})
target_include_directories(${PROJECT_NAME} PRIVATE ${INCLUDE_DIRS})
target_link_directories(${PROJECT_NAME} PRIVATE ${MPV_LIBRARY_DIRS})
target_link_libraries(${PROJECT_NAME} PRIVATE ${LINK_LIBS})
target_compile_definitions(${PROJECT_NAME} PRIVATE APP_VERSION="${GIT_VERSION}")
if(USE_OPENGL_ES3)
  target_compile_definitions(${PROJECT_NAME} PRIVATE IMGUI_IMPL_OPENGL_ES3)
endif()
if(USE_PATCHED_GLFW)
  target_compile_definitions(${PROJECT_NAME} PRIVATE GLFW_PATCHED)
endif()


if(WIN32)
  set_target_properties( ${PROJECT_NAME} PROPERTIES  VS_DEBUGGER_WORKING_DIRECTORY "$<TARGET_FILE_DIR:${PROJECT_NAME}>")
  set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME})
 file(COPY ${MPV_DLL} DESTINATION ${CMAKE_BINARY_DIR}/Release)
 file(COPY ${MPV_DLL} DESTINATION ${CMAKE_BINARY_DIR}/Debug)
endif()

if(CREATE_PACKAGE)
  include(CreateCpackPackage)
  prepare_package()
  create_package()
else()
  include(GNUInstallDirs)
  install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
  if(UNIX AND NOT APPLE)
    install(FILES ${PROJECT_SOURCE_DIR}/resources/linux/implay.desktop DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/applications)
    install(FILES ${PROJECT_SOURCE_DIR}/resources/icon.png DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pixmaps RENAME implay.png)
  endif()
endif()
