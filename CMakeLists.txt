cmake_minimum_required(VERSION 3.10 FATAL_ERROR)
project(libdxfrw VERSION 0.6.3)

set(WIN_RUNTIME "MT" CACHE STRING "Windows runtime")
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()

if(WIN32)
    message(STATUS "Configuring with runtime ${WIN_RUNTIME}")
    if(WIN_RUNTIME STREQUAL "MT")
        set(CMAKE_CXX_FLAGS_RELEASE "-MT")
        set(CMAKE_CXX_FLAGS_DEBUG "-MTd")
    elseif(WIN_RUNTIME STREQUAL "MD")
        set(CMAKE_CXX_FLAGS_RELEASE "-MD")
        set(CMAKE_CXX_FLAGS_DEBUG "-MDd")
    endif()
endif()

# Enable testing
enable_testing()

file(GLOB libdxfrw_sources src/*.cpp)
file(GLOB libdxfrw_headers include/*.h)
file(GLOB libdxfrw_intern_sources src/intern/*.cpp)

# Find iconv library
if(WIN32)
    message(STATUS "CMAKE_SOURCE_DIR = ${CMAKE_SOURCE_DIR}")
    include_directories( PRIVATE ${CMAKE_SOURCE_DIR}/include)
    # include_directories(${CMAKE_SOURCE_DIR}/../LightRightApp/core/template/iconv.h)
    # link_directories(${CMAKE_SOURCE_DIR}/packages/libiconv.1.14.0.11/build/native/lib)
    set(ICONV_LIBRARY iconv)
else()
    # On Unix-like systems (Linux, macOS)
    find_package(Iconv REQUIRED)
    if(Iconv_FOUND)
        include_directories(${Iconv_INCLUDE_DIRS})
        set(ICONV_LIBRARY ${Iconv_LIBRARIES})
    else()
        # Fallback: try to link against iconv directly
        set(ICONV_LIBRARY iconv)
    endif()
endif()

include_directories(include)

add_library(dxfrw STATIC ${libdxfrw_sources} ${libdxfrw_intern_sources})
target_compile_definitions(dxfrw PUBLIC LIBICONV_PLUG)
if(NOT WIN32)
    target_link_libraries(dxfrw PUBLIC ${ICONV_LIBRARY})
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

# Tests (not built on Windows: iconv is provided by the consuming project)
if(NOT WIN32)
add_executable(test_basic tests/test_basic.cpp)
target_include_directories(test_basic PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_basic dxfrw)
add_test(NAME BasicTests COMMAND test_basic)

add_executable(test_entities tests/test_entities.cpp)
target_include_directories(test_entities PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_entities dxfrw)
add_test(NAME EntityTests COMMAND test_entities)

add_executable(test_polylines tests/test_polylines.cpp)
target_include_directories(test_polylines PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_polylines dxfrw)
add_test(NAME PolylineTests COMMAND test_polylines)

add_executable(test_text tests/test_text.cpp)
target_include_directories(test_text PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_text dxfrw)
add_test(NAME TextTests COMMAND test_text)

add_executable(test_tables tests/test_tables.cpp)
target_include_directories(test_tables PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_tables dxfrw)
add_test(NAME TableTests COMMAND test_tables)

add_executable(test_blocks tests/test_blocks.cpp)
target_include_directories(test_blocks PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_blocks dxfrw)
add_test(NAME BlockTests COMMAND test_blocks)

add_executable(test_versions tests/test_versions.cpp)
target_include_directories(test_versions PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_versions dxfrw)
add_test(NAME VersionTests COMMAND test_versions)

add_executable(test_errors tests/test_errors.cpp)
target_include_directories(test_errors PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_errors dxfrw)
add_test(NAME ErrorTests COMMAND test_errors)

add_executable(test_dimensions tests/test_dimensions.cpp)
target_include_directories(test_dimensions PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_dimensions dxfrw)
add_test(NAME DimensionTests COMMAND test_dimensions)

add_executable(test_annotations tests/test_annotations.cpp)
target_include_directories(test_annotations PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_annotations dxfrw)
add_test(NAME AnnotationTests COMMAND test_annotations)

add_executable(test_attributes tests/test_attributes.cpp)
target_include_directories(test_attributes PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_attributes dxfrw)
add_test(NAME AttributeTests COMMAND test_attributes)

add_executable(test_class_underflow tests/test_class_underflow.cpp)
target_include_directories(test_class_underflow PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_class_underflow dxfrw)
add_test(NAME ClassUnderflowTests COMMAND test_class_underflow)

add_executable(test_dwg_classes tests/test_dwg_classes.cpp)
target_include_directories(test_dwg_classes PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/tests)
target_link_libraries(test_dwg_classes dxfrw)
add_test(NAME DWGClassParsingTests COMMAND test_dwg_classes)
endif()
