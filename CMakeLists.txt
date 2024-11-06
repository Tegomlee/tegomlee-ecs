cmake_minimum_required(VERSION 3.12)

project(tego-ecs VERSION 1.0.0)

# Set the standard to C++
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Option to build as static or shared library
option(TEGO_BUILD_SHARED "build tego-ecs as a shared library" ON)

# Collect all .cpp files in the src directory
file(GLOB_RECURSE SOURCES "${CMAKE_SOURCE_DIR}/src/*.cpp")

# Choose library type based on the TEGO_BUILD_SHARED option
if(TEGO_BUILD_SHARED)
    if(MSVC)
        add_compile_options(
            $<$<CONFIG:>:/MD> #---------|
            $<$<CONFIG:Debug>:/MDd> #---|-- Dynamically link the runtime libraries
            $<$<CONFIG:Release>:/MD> #--|
        )
    endif()

    add_library("${CMAKE_PROJECT_NAME}" SHARED ${SOURCES})
    target_compile_definitions("${CMAKE_PROJECT_NAME}" PRIVATE "TEGO_EXPORT") # Define for shared export
else()
    if(MSVC)
        add_compile_options(
            $<$<CONFIG:>:/MT> #---------|
            $<$<CONFIG:Debug>:/MTd> #---|-- Statically link the runtime libraries
            $<$<CONFIG:Release>:/MT> #--|
        )
    endif()

    add_library("${CMAKE_PROJECT_NAME}" STATIC ${SOURCES})
    target_compile_definitions("${CMAKE_PROJECT_NAME}" PRIVATE "TEGO_STATIC") # Define for static library

    # Set output name for static library
    set_target_properties("${CMAKE_PROJECT_NAME}" PROPERTIES OUTPUT_NAME "tego-ecs-s")
endif()

# Include directories
target_include_directories("${CMAKE_PROJECT_NAME}" PUBLIC "${CMAKE_SOURCE_DIR}/include")

# Specify the output directory for the library
set_target_properties("${CMAKE_PROJECT_NAME}" PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)

# Optionally, add preprocessor definitions based on the build type
if(DEFINED TEGO_EXPORT)
    message(STATUS "Building as a shared library, exporting symbols.")
endif()

if(DEFINED TEGO_STATIC)
    message(STATUS "Building as a static library.")
endif()
