cmake_minimum_required(VERSION 3.9)

set(SCOPY_MODULE newplugin)

message(STATUS "building plugin: " ${SCOPY_MODULE})

project(scopy-${SCOPY_MODULE} VERSION 0.1 LANGUAGES CXX) 

set(PLUGIN_DISPLAY_NAME NEW PLUGIN)
set(PLUGIN_DESCRIPTION A NEW PLUGIN)

include(GenerateExportHeader) 

# TODO: split stylesheet/resources and add here TODO: export header files correctly

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON) 

set(CMAKE_AUTOUIC_SEARCH_PATHS ${CMAKE_CURRENT_SOURCE_DIR}/ui) 
set(CMAKE_AUTOUIC ON) 
set(CMAKE_AUTOMOC ON) 
set(CMAKE_AUTORCC ON) 

set(CMAKE_INCLUDE_CURRENT_DIR ON) 

set(CMAKE_CXX_VISIBILITY_PRESET hidden) 
set(CMAKE_VISIBILITY_INLINES_HIDDEN TRUE) 

file(GLOB SRC_LIST src/*.cpp src/*.cc) 
file(GLOB HEADER_LIST include/${SCOPY_MODULE}/*.h include/${SCOPY_MODULE}/*.hpp) 
file(GLOB UI_LIST ui/*.ui) 

set(ENABLE_TESTING ON)
if(ENABLE_TESTING) 
    add_subdirectory(test) 
endif() 

set(PROJECT_SOURCES ${SRC_LIST} ${HEADER_LIST} ${UI_LIST}) 
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS REQUIRED Widgets Core) 

qt_add_resources(PROJECT_RESOURCES res/resources.qrc) 
add_library(${PROJECT_NAME} SHARED ${PROJECT_SOURCES} ${PROJECT_RESOURCES}) 

generate_export_header( 
    ${PROJECT_NAME} EXPORT_FILE_NAME ${CMAKE_CURRENT_SOURCE_DIR}/include/${SCOPY_MODULE}/${PROJECT_NAME}_export.h 
)

message("Generate plugin style:")
include(ScopyStyle)
file(MAKE_DIRECTORY ${SCOPY_PACKAGE_BUILD_PATH}/${PACKAGE_NAME}/style/qss)
generate_style("--plugin" ${CMAKE_CURRENT_SOURCE_DIR}/style  ${CMAKE_CURRENT_SOURCE_DIR}/include/${SCOPY_MODULE} ${SCOPY_PACKAGE_BUILD_PATH}/${PACKAGE_NAME})

configure_file(
	include/${SCOPY_MODULE}/scopy-${SCOPY_MODULE}_config.h.cmakein
	${CMAKE_CURRENT_SOURCE_DIR}/include/${SCOPY_MODULE}/scopy-${SCOPY_MODULE}_config.h @ONLY
)

target_include_directories(${PROJECT_NAME} INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include) 
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include/${SCOPY_MODULE}) 

target_include_directories(${PROJECT_NAME} PUBLIC ${CURRENT_PKG_PATH}/include)
target_include_directories(${PROJECT_NAME} PUBLIC scopy-pluginbase scopy-gui) 

target_link_libraries( 
    ${PROJECT_NAME} 
    PUBLIC Qt::Widgets 
        Qt::Core 
        scopy-pluginbase 
        scopy-gui 
        scopy-iioutil 
        scopy-iio-widgets
)


if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
	configureinstallersettings(${SCOPY_MODULE} ${PLUGIN_DESCRIPTION} FALSE)
endif()

set(NEWPLUGIN_TARGET_NAME ${PROJECT_NAME} PARENT_SCOPE)