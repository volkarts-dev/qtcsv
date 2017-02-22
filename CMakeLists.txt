cmake_minimum_required (VERSION 3.1)

project (qtcsv)

#define names
set(LIB_MAJOR_VERSION 1)
set(LIB_MINOR_VERSION 3)
set(LIB_REVISION_VERSION 1)
set(LIB_VERSION ${LIB_MAJOR_VERSION}.${LIB_MINOR_VERSION}.${LIB_REVISION_VERSION})

#add additional targets enabling sanitizer on demand
option(SHARED_LIB "build as shared lib" ON) # ON is the default
option(STATIC_LIB "build as static lib" OFF) # OFF is the default
option(BUILD_TESTS "build tests" ON) # ON is the default
option(USE_QT4 "builds against Qt4 if true, otherwise builds against Qt5" OFF)# OFF ist the default

if(USE_QT4)
	find_package(Qt4 REQUIRED)
	set(QT_CORE_TARGET Qt4::QtGui)
else()
	find_package(Qt5Core REQUIRED)
	set(QT_CORE_TARGET Qt5::Core)
endif(USE_QT4)

#define names
set(LIBRARY_NAME ${PROJECT_NAME})

file(GLOB_RECURSE SOURCE_FILES ${CMAKE_CURRENT_SOURCE_DIR}/sources/*.cpp)

if(SHARED_LIB)
	set(LIBRARY_NAME_SHARED ${LIBRARY_NAME})
	add_library(${LIBRARY_NAME_SHARED} SHARED ${SOURCE_FILES})
	#include root project folder as private, because the source headers are included with source/*.h
	target_include_directories(${LIBRARY_NAME_SHARED} PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include> $<INSTALL_INTERFACE:include> PRIVATE .)
	set_target_properties(${LIBRARY_NAME_SHARED} PROPERTIES VERSION ${LIB_VERSION} SOVERSION ${LIB_MAJOR_VERSION}.${LIB_MINOR_VERSION})
	TARGET_LINK_LIBRARIES(${LIBRARY_NAME_SHARED} PRIVATE ${QT_CORE_TARGET})
endif(SHARED_LIB)
if(STATIC_LIB)
	set(LIBRARY_NAME_STATIC ${LIBRARY_NAME}_static)
	add_library(${LIBRARY_NAME_STATIC} STATIC ${SOURCE_FILES})
	#include root project folder as private, because the source headers are included with source/*.h
  target_include_directories(${LIBRARY_NAME_STATIC} PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include> $<INSTALL_INTERFACE:include> PRIVATE .)
	#set static lib name to the same of shared lib
	set_target_properties(${LIBRARY_NAME_STATIC} PROPERTIES OUTPUT_NAME ${LIBRARY_NAME})
	TARGET_LINK_LIBRARIES(${LIBRARY_NAME_STATIC} PRIVATE ${QT_CORE_TARGET})
endif(STATIC_LIB)

install(TARGETS ${LIBRARY_NAME_SHARED} ${LIBRARY_NAME_STATIC} EXPORT ${LIBRARY_NAME}Config
        LIBRARY DESTINATION lib
				ARCHIVE DESTINATION lib)
install(DIRECTORY include DESTINATION .)

#create and install cmake package files
install(EXPORT ${LIBRARY_NAME}Config DESTINATION share/${LIBRARY_NAME}/cmake)
export(TARGETS ${LIBRARY_NAME_SHARED} ${LIBRARY_NAME_STATIC} FILE ${LIBRARY_NAME}Config.cmake)

if(BUILD_AND_RUN_TESTS)
	add_subdirectory(tests)
endif(BUILD_AND_RUN_TESTS)