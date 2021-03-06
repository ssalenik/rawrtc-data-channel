# Project
cmake_minimum_required(VERSION 3.2)
# TODO: Find a way to keep this in sync with the one in rawrtcdc.h
project(librawrtcdc
        VERSION 0.0.1)
set(PROJECT_AUTHOR
        "Lennart Grahl <lennart.grahl@gmail.com>")
set(PROJECT_DESCRIPTION
        "A standalone WebRTC and ORTC data channel implementation.")
set(PROJECT_URL
        "https://github.com/rawrtc/rawrtc-data-channel")

# Use prefix pkgconfig
set(ENV{PKG_CONFIG_PATH} "${CMAKE_INSTALL_PREFIX}/lib/pkgconfig/:$ENV{PKG_CONFIG_PATH}")

# Dependency versions
set(LIB_RE_VERSION "libre >= 0.5.7")
set(LIB_RAWRTCC_VERSION "librawrtcc >= 0.0.1")
# Note: This MUST be here as cmake 3.5.0 overwrites LIB_RE_VERSION for some reason...
set(DEPENDENCY_VERSIONS "${LIB_RE_VERSION}, ${LIB_RAWRTCC_VERSION}")

# Debug build type as default
if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type selected, using DEBUG")
    set(CMAKE_BUILD_TYPE "DEBUG")
endif ()

# Enable verbose output in DEBUG mode
if (${CMAKE_BUILD_TYPE} MATCHES "DEBUG")
    message(STATUS "enabling verbose outout")
    set(CMAKE_VERBOSE_MAKEFILE on)
endif ()

# Flag for enabling building of SCTP redirect
option(SCTP_REDIRECT_TRANSPORT "Build the SCTP redirect transport." OFF)

# Use pkg-config
find_package(PkgConfig REQUIRED)

# Dependency list
set(rawrtcdc_DEP_LIBRARIES)

# Apple-specific dependencies
# TODO: This should probably be in libre's pkg-config file in Libs.private
if (APPLE)
    find_library(LIB_SYSTEM_CONFIGURATION_LIBRARIES NAMES SystemConfiguration)
    find_library(LIB_CORE_FOUNDATION_LIBRARIES NAMES CoreFoundation)
    list(APPEND rawrtcdc_DEP_LIBRARIES
            ${LIB_SYSTEM_CONFIGURATION_LIBRARIES}
            ${LIB_CORE_FOUNDATION_LIBRARIES})
endif ()

# Dependency: libre
pkg_check_modules(LIB_RE REQUIRED ${LIB_RE_VERSION})
include_directories(${LIB_RE_STATIC_INCLUDE_DIRS} ${LIB_RE_STATIC_INCLUDEDIR})
link_directories(${LIB_RE_STATIC_LIBRARY_DIRS})
list(APPEND rawrtcdc_DEP_LIBRARIES ${LIB_RE_STATIC_LIBRARIES})

# Dependency: usrsctp
# TODO: Use the pkg-config file of usrsctp once it has been added
find_library(LIB_USRSCTP_LIBRARIES NAMES usrsctp.a libusrsctp.a HINTS ${CMAKE_INSTALL_PREFIX}/lib )
find_path(LIB_USRSCTP_INCLUDE_DIRS usrsctp.h HINTS ${CMAKE_INSTALL_PREFIX}/include )
get_filename_component(LIB_USRSCTP_LIBRARY_DIRS ${LIB_USRSCTP_LIBRARIES} DIRECTORY)
include_directories(${LIB_USRSCTP_INCLUDE_DIRS})
link_directories(${LIB_USRSCTP_LIBRARY_DIRS})
list(APPEND rawrtcdc_DEP_LIBRARIES ${LIB_USRSCTP_LIBRARIES})

# Dependency: librawrtcc
pkg_check_modules(LIB_RAWRTCC REQUIRED ${LIB_RAWRTCC_VERSION})
include_directories(${LIB_RAWRTCC_STATIC_INCLUDE_DIRS} ${LIB_RAWRTCC_INCLUDEDIR})
link_directories(${LIB_RAWRTCC_STATIC_LIBRARY_DIRS})
list(APPEND rawrtcdc_DEP_LIBRARIES ${LIB_RAWRTCC_STATIC_LIBRARIES})

# Dependency versions
set(PKG_CONFIG_REQUIRES "")
set(PKG_CONFIG_REQUIRES_PRIVATE "${DEPENDENCY_VERSIONS}")

# Linker flags (includes libs that have no pkg-config files)
set(PKG_CONFIG_LIB_DIRS "-L\${libdir}")
set(PKG_CONFIG_LIBRARIES "-lrawrtcdc")
set(PKG_CONFIG_LIBS "${PKG_CONFIG_LIB_DIRS} ${PKG_CONFIG_LIBRARIES}")
set(PKG_CONFIG_LIB_DIRS_PRIVATE "${PKG_CONFIG_LIB_DIRS} -L${LIB_USRSCTP_LIBRARY_DIRS}")
set(PKG_CONFIG_LIBRARIES_PRIVATE "${PKG_CONFIG_LIBRARIES} ${LIB_USRSCTP_LIBRARIES}")
set(PKG_CONFIG_LIBS_PRIVATE "${PKG_CONFIG_LIB_DIRS_PRIVATE} ${PKG_CONFIG_LIBRARIES_PRIVATE}")

# Cflags (includes includes that have no pkg-config files)
set(PKG_CONFIG_CFLAGS "-I\${includedir} -I${LIB_USRSCTP_INCLUDE_DIRS}")

# Add custom target to install the library
add_custom_target(install-${PROJECT_NAME}
        $(MAKE) install
        COMMENT "Installing ${PROJECT_NAME}")

# Walk through subdirectories
add_subdirectory(docs)
add_subdirectory(src)
