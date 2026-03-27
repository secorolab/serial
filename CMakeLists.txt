cmake_minimum_required(VERSION 3.25)
project(serial VERSION 1.0.0 LANGUAGES CXX)

# Options
option(BUILD_SHARED_LIBS "Build shared libraries" ON)

# C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Platform-specific libs
if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
elseif(UNIX)
    set(PLATFORM_LIBS rt pthread)
else()
    set(PLATFORM_LIBS setupapi)
endif()

# Sources
set(serial_SRCS
    src/serial.cc
)

set(serial_HEADERS
    include/serial/serial.h
    include/serial/v8stdint.h
)

if(APPLE)
    list(APPEND serial_SRCS
        src/impl/unix.cc
        src/impl/list_ports/list_ports_osx.cc
    )
elseif(UNIX)
    list(APPEND serial_SRCS
        src/impl/unix.cc
        src/impl/list_ports/list_ports_linux.cc
    )
else()
    list(APPEND serial_SRCS
        src/impl/win.cc
        src/impl/list_ports/list_ports_win.cc
    )
endif()

# Library
add_library(serial ${serial_SRCS} ${serial_HEADERS})

target_include_directories(serial
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
)

# Link platform libs
if(APPLE)
    target_link_libraries(serial PRIVATE ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    target_link_libraries(serial PRIVATE ${PLATFORM_LIBS})
else()
    target_link_libraries(serial PRIVATE ${PLATFORM_LIBS})
endif()

# Example executable
add_executable(serial_example examples/serial_example.cc)
target_link_libraries(serial_example serial)

# Install and Export
include(GNUInstallDirs)

install(TARGETS serial
    EXPORT serialTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Export targets
install(EXPORT serialTargets
    FILE serialTargets.cmake
    NAMESPACE serial::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/serial
)

# Generate config files
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/serialConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/serialConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/serialConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/serial
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/serialConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/serialConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/serial
)
