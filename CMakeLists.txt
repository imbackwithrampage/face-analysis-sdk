# -*-cmake-*-
PROJECT(CSIRO-FaceAnalysis-SDK)

cmake_minimum_required(VERSION 2.8)

#set(CMAKE_VERBOSE_MAKEFILE true)

# Configurable options
OPTION(WITH_GUI "Build the GUI" OFF)


# Third party libraries
find_package(OpenCV REQUIRED core highgui imgproc objdetect)

IF(APPLE)
  find_library(FOUNDATION Foundation)
  SET(EXTRA_LIBS ${FOUNDATION})
ENDIF()

IF(UNIX AND NOT APPLE)
  FIND_PROGRAM(HAVE_GNUSTEP NAMES gnustep-config)
  IF(HAVE_GNUSTEP)
    SET(WITH_GUI "Build the GUI" ON)
    MESSAGE(STATUS "Found GnuStep")
  ELSE(HAVE_GNUSTEP)
    MESSAGE(STATUS "Cannot find GnuStep")
  ENDIF(HAVE_GNUSTEP)
ENDIF(UNIX AND NOT APPLE)


# Third party programs
if(NOT DEFINED FFMPEG)
  find_program(FFMPEG ffmpeg)
endif()

if (NOT DEFINED BASH)
  find_program(BASH bash)
endif()

# C++ Preprocessor definitions
ADD_DEFINITIONS(-D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE)
ADD_DEFINITIONS(-D_WITH_AVATAR_)

# Compiling and linking
# - type
set(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "Configs" FORCE)
if(DEFINED CMAKE_BUILD_TYPE AND CMAKE_VERSION VERSION_GREATER "2.8")
   set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS ${CMAKE_CONFIGURATION_TYPES})
endif()
SET(CMAKE_BUILD_TYPE Release CACHE STRING "")

# - flags
if(NOT WIN32)
    SET(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS} -g -O0  -Wall -Wextra -DDEBUG")
    SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -O0 -g  -Wall -Wextra -DDEBUG")

    SET(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS} -O3  -Wall -Wextra")
    SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -Wall -O3  -Wextra")
endif()

SET(LIBS ${OpenCV_LIBS} ${EXTRA_LIBS})
INCLUDE_DIRECTORIES(${OpenCV_INCLUDE_DIRS})

# Build paths
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lib")
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lib")
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/bin")

# Include directories
INCLUDE_DIRECTORIES("src/")
INCLUDE_DIRECTORIES("src/avatar/")
INCLUDE_DIRECTORIES("${CMAKE_CURRENT_BINARY_DIR}/src/")
INCLUDE_DIRECTORIES(OpenCV_INCLUDE_DIRS)


# Subdirectories with CMakeLists.txt
ADD_SUBDIRECTORY(src/utils)
ADD_SUBDIRECTORY(src/tracker)
ADD_SUBDIRECTORY(src/avatar)
ADD_SUBDIRECTORY(src/face-fit)
ADD_SUBDIRECTORY(src/test)
ADD_SUBDIRECTORY(src/scripts)
ADD_SUBDIRECTORY(src/map-list)
ADD_SUBDIRECTORY(src/change-pathnames)
ADD_SUBDIRECTORY(src/expression-transfer)
ADD_SUBDIRECTORY(src/display-avatar)
ADD_SUBDIRECTORY(src/display-tracking)
ADD_SUBDIRECTORY(src/create-avatar-model)
ADD_SUBDIRECTORY(src/display-3d-points)

# install(EXPORT CSIRO DESTINATION ${CMAKE_INSTALL_PREFIX})
install(FILES src/CSIROConfig.cmake DESTINATION "${CMAKE_INSTALL_PREFIX}")

IF(WITH_GUI)
  SET(CPACK_PACKAGE_NAME "clm")
  SET(CPACK_PACKAGE_VENDOR "CI2CV-CSIRO ci2cv.net")
  SET(CPACK_PACKAGE_VERSION "0.9.1")
  SET(CPACK_PACKAGE_VERSION_MAJOR "0")
  SET(CPACK_PACKAGE_VERSION_MINOR "9")
  SET(CPACK_PACKAGE_VERSION_PATCH "1")

  set(CPACK_BINARY_DRAGNDROP ON)
  SET(CPACK_BINARY_PACKAGEMAKER OFF)
  SET(CPACK_SOURCE_TGZ OFF)
  ADD_SUBDIRECTORY(src/qt-gui)
  MESSAGE(STATUS "Will build the GUI")
else()
  MESSAGE(STATUS "Missing GUI components - will NOT build the GUI")
endif()

#cpack stuff goes here
