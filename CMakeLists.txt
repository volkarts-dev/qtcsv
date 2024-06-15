cmake_minimum_required(VERSION 3.13)

project(qtcsv VERSION 1.7.0 LANGUAGES CXX)

# set options
option(QTCSV_STATIC_LIB "Build as static lib if ON, otherwise build shared lib" OFF)
option(QTCSV_BUILD_TESTS "Build tests" ON)

# find qt package
find_package(Qt6 COMPONENTS Core REQUIRED)

# instruct CMake to run moc automatically when needed.
set(CMAKE_AUTOMOC ON)

if(QTCSV_STATIC_LIB)
    add_library(${PROJECT_NAME} STATIC "")
    #propogate the QTCSV_STATIC_LIB define for all qtcsv consuming stuff
    #necessary for correct QTCSVSHARED_EXPORT handling, espacially for windows (cross compilation) builds
    target_compile_definitions(${PROJECT_NAME} PUBLIC -DQTCSV_STATIC_LIB)
else()
    add_library(${PROJECT_NAME} SHARED "")
    set_target_properties(${PROJECT_NAME} PROPERTIES
        VERSION ${PROJECT_VERSION}
        SOVERSION ${PROJECT_VERSION_MAJOR})
endif()

# set list of source files
target_sources(${PROJECT_NAME} PRIVATE
    include/qtcsv/abstractdata.h
    include/qtcsv/qtcsv_global.h
    include/qtcsv/reader.h
    include/qtcsv/stringdata.h
    include/qtcsv/symbols.h
    include/qtcsv/variantdata.h
    include/qtcsv/writer.h
    sources/contentiterator.cpp
    sources/contentiterator.h
    sources/filechecker.h
    sources/reader.cpp
    sources/stringdata.cpp
    sources/variantdata.cpp
    sources/writer.cpp
)

#use the QTCSV_LIBRARY define only for qtcsv build time to use export statements instead of import statements
target_compile_definitions(${PROJECT_NAME} PRIVATE
    QTCSV_LIBRARY
    QT_NO_CAST_FROM_ASCII
    QT_NO_CAST_TO_ASCII
)

# include root project folder as private, because the source headers are included with source/*.h
target_include_directories(${PROJECT_NAME}
    PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include> $<INSTALL_INTERFACE:include>
    PRIVATE .
)

# set compiler flags for the library target
if(CMAKE_COMPILER_IS_GNUCXX)
    target_compile_options(${PROJECT_NAME} PRIVATE
        -Wall -Werror -Wformat=2
        -Wuninitialized -Winit-self -Wswitch-enum -Wundef
        -Wpointer-arith -Wdisabled-optimization -Wcast-align -Wcast-qual
    )
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE Qt6::Core)

install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}Config
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

install(DIRECTORY include DESTINATION .)

# create and install cmake package files
install(EXPORT ${PROJECT_NAME}Config DESTINATION share/${PROJECT_NAME}/cmake)
export(TARGETS ${PROJECT_NAME} FILE ${PROJECT_NAME}Config.cmake)

if(QTCSV_BUILD_TESTS)
    add_subdirectory(tests)
endif()
