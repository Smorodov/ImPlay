cmake_minimum_required(VERSION 3.13)
project(imgui)

set(CMAKE_CXX_STANDARD 17)

find_package(Freetype REQUIRED)

add_library(imgui STATIC
  source/imgui.cpp
  source/imgui_demo.cpp
  source/imgui_draw.cpp
  source/imgui_freetype.cpp
  source/imgui_impl_glfw.cpp
  source/imgui_impl_opengl3.cpp
  source/imgui_tables.cpp
  source/imgui_widgets.cpp
  source/fonts/fontawesome.cpp
  source/fonts/source_code_pro.cpp
  source/fonts/unifont.cpp
)

target_include_directories(imgui PUBLIC include ${FREETYPE_INCLUDE_DIRS} ${GLFW_INCLUDE_DIRS})
target_link_libraries(imgui PUBLIC ${CMAKE_DL_LIBS} ${FREETYPE_LIBRARIES} ${GLFW_LIBRARIES})
if(USE_OPENGL_ES3)
  target_compile_definitions(imgui PRIVATE IMGUI_IMPL_OPENGL_ES3)
  target_link_libraries(imgui PUBLIC ${OPENGL_LIBRARIES})
endif()