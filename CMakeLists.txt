#cmake_minimum_required (VERSION 2.6)
cmake_minimum_required(VERSION 3.1)
cmake_policy(SET CMP0079 NEW)

project (libblinkstick LANGUAGES C)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
include(GNUInstallDirs)

option(BUILD_DOCS "Build documentation" OFF)
option(BUILD_CLI "Build command line BlinkStick control program" ON)

if(WIN32)
  list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
  # find_package(HIDAPI REQUIRED)
else(WIN32)
  find_package(PkgConfig REQUIRED)
  # pkg_search_module(HIDAPI REQUIRED hidapi)
endif(WIN32)

# br
include_directories(hidapi)
include_directories(libblinkstick)
find_package(PkgConfig REQUIRED)
pkg_search_module(HIDAPI_LIBUSB REQUIRED hidapi-libusb)


set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/target)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/target)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/target)

if(BUILD_DOCS)
  find_package(Doxygen)
  if (DOXYGEN_FOUND)
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile @ONLY)
    message("Doxygen found, setting build rules")

    add_custom_target(docs
    ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Generating API documentation with Doxygen" VERBATIM
    )
  else()
    message("Doxygen needs to be installed to generate the doxygen documentation")
  endif (DOXYGEN_FOUND)
endif(BUILD_DOCS)


set(INSTALL_BIN_DIR "${CMAKE_INSTALL_PREFIX}/bin" CACHE PATH "Installation directory for executables")
set(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
set(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "Installation directory for headers")
set(INSTALL_MAN_DIR "${CMAKE_INSTALL_PREFIX}/share/man" CACHE PATH "Installation directory for manual pages")
set(INSTALL_PKGCONFIG_DIR "${CMAKE_INSTALL_PREFIX}/share/pkgconfig" CACHE PATH "Installation directory for pkgconfig (.pc) files")

configure_file( ${CMAKE_CURRENT_SOURCE_DIR}/libblinkstick.pc.in
		${CMAKE_BINARY_DIR}/libblinkstick.pc @ONLY)

install(FILES ${CMAKE_BINARY_DIR}/libblinkstick.pc DESTINATION "${INSTALL_PKGCONFIG_DIR}")

set(libblinkstick_VERSION_MAJOR 1)
set(libblinkstick_VERSION_MINOR 0)
set(libblinkstick_VERSION_PATCH 0)

set(libblinkstick_VERSION "${libblinkstick_VERSION_MAJOR}.${libblinkstick_VERSION_MINOR}.${libblinkstick_VERSION_PATCH}")
add_subdirectory(src)

target_compile_options(libblinkstick PRIVATE ${HIDAPI_LIBUSB_CFLAGS})
target_link_libraries(libblinkstick ${HIDAPI_LIBUSB_LDFLAGS})

