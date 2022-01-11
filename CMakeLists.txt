cmake_minimum_required(VERSION 3.15 FATAL_ERROR)
project(libdxfrw)

set(WIN_RUNTIME "MT" CACHE STRING "Windows runtime")

if (WIN32)
    message("Configuring with runtime " ${WIN_RUNTIME})
    if (${WIN_RUNTIME} STREQUAL "MT")
        set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
        message("Setting to MultiThreaded")
    elseif(${WIN_RUNTIME} STREQUAL "MD")
        set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL")
        message("Setting to MultiThreaded-DLL")
    endif()
endif()


file(GLOB libdxfrw_sources src/*.cpp)
file(GLOB libdxfrw_headers include/*.h)
file(GLOB libdxfrw_intern_sources src/intern/*.cpp)


add_library(dxfrw STATIC ${libdxfrw_sources} ${libdxfrw_intern_sources})
target_include_directories(dxfrw PUBLIC include)

if(WIN32)
#target_include_directories(dxfrw PRIVATE packages/libiconv.1.14.0.11/build/native/include)
#target_link_directories(dxfrw PRIVATE packages/libiconv.1.14.0.11/build/native/lib)
endif()


install(FILES ${libdxfrw_headers} DESTINATION include)

if(WIN32)
  install(TARGETS dxfrw
          CONFIGURATIONS Debug
          LIBRARY DESTINATION Debug/lib
          ARCHIVE DESTINATION Debug/lib)
  install(TARGETS dxfrw
          CONFIGURATIONS Release
          LIBRARY DESTINATION Release/lib
          ARCHIVE DESTINATION Release/lib)
else()
  install(TARGETS dxfrw
          LIBRARY DESTINATION lib
          ARCHIVE DESTINATION lib)
endif()