cmake_minimum_required(VERSION 2.8.8)

# ---[ Configurations types
set(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "Possible configurations" FORCE)
if (DEFINED CMAKE_BUILD_TYPE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS ${CMAKE_CONFIGURATION_TYPES})
endif()

# ---[ Solution name
project(kfusion C CXX)

# ---[ utility
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/Modules/")
include(cmake/Utils.cmake)
include(cmake/Targets.cmake)

# ---[ find dependencies
find_package( OpenCV 2.4.8 REQUIRED COMPONENTS core viz highgui HINTS "$ENV{USERPROFILE}/Documents/GitHub/BUILDS/OpenCV" "C:/Users/Public/Documents/opencv/build") 
	
string(REGEX REPLACE "/Release/Release" "/Release" OpenCV_LIB_DIR_OPT "${OpenCV_LIB_DIR_OPT}")
string(REGEX REPLACE "/Debug/Debug" "/Debug" OpenCV_LIB_DIR_DBG "${OpenCV_LIB_DIR_DBG}")

IF(OpenCV_FOUND AND ("${OpenCV_DIR}" STREQUAL "${OpenCV_CONFIG_PATH}") AND (NOT("${OpenCV_LIB_DIR_DBG}" STREQUAL "/Debug")) AND (NOT("${OpenCV_LIB_DIR_OPT}" STREQUAL "/Release")) )
	message(STATUS "OpenCV Found!")
	string(REGEX REPLACE "/lib" "/bin" OpenCV_BIN_DIR_OPT "${OpenCV_LIB_DIR_OPT}")
	string(REGEX REPLACE "/lib" "/bin" OpenCV_BIN_DIR_DBG "${OpenCV_LIB_DIR_DBG}")
ELSEIF(EXISTS "${OpenCV_DIR}/")
	message(WARNING "OpenCV not entirely found, but directory address <OpenCV_DIR> located and so assuming local build at ${OpenCV_DIR}")
	
	IF( (NOT EXISTS "${OpenCV_DIR}/lib/Debug/") AND (NOT EXISTS "${OpenCV_DIR}/lib/Release/") )
		MESSAGE(FATAL_ERROR "Provided path for OpenCV does not appear to contain valid debug OR release libraries!")
	ENDIF()
	
	SET(OpenCV_LIB_DIR_OPT "${OpenCV_DIR}/lib/Release" CACHE STRING "..." FORCE)
	SET(OpenCV_LIB_DIR_DBG "${OpenCV_DIR}/lib/Debug" CACHE STRING "..." FORCE)
	SET(OpenCV_3RDPARTY_LIB_DIR_OPT "${OpenCV_DIR}/lib/Release" CACHE STRING "..." FORCE)
	SET(OpenCV_3RDPARTY_LIB_DIR_DBG "${OpenCV_DIR}/lib/Debug" CACHE STRING "..." FORCE)
	
	LIST(APPEND _OpenCV_LIB_PATH "${OpenCV_DIR}/bin/Debug")
	LIST(APPEND _OpenCV_LIB_PATH "${OpenCV_DIR}/bin/Release")
	
	SET(OPENCV_FOUND TRUE CACHE BOOL "..." FORCE)
	SET(OpenCV_FOUND TRUE CACHE BOOL "..." FORCE)
	
	SET(OpenCV_BIN_DIR_OPT "${OpenCV_DIR}/bin/Release")
	SET(OpenCV_BIN_DIR_DBG "${OpenCV_DIR}/bin/Debug")
	SET(OpenCV_CONFIG_PATH "${OpenCV_DIR}" CACHE STRING "..." FORCE)
ELSE()
	MESSAGE(FATAL_ERROR "OpenCV not found! Please set the <OpenCV_DIR> variable to the OpenCV build directory.")
ENDIF()

IF(OpenCV_FOUND)
	include_directories(${OpenCV_INCLUDE_DIRS})
	link_directories(${_OpenCV_LIB_PATH})
	STRING(REGEX REPLACE "\\." "" OPENCV_VER "${OpenCV_VERSION}")
ENDIF()

find_package(CUDA 5.0 REQUIRED)

SET(OPENNI2_LIBRARY "C:/Program Files/OpenNI2/Lib/OpenNI2.lib" CACHE STRING "..." FORCE)
SET(OPENNI2_INCLUDE_DIRS "C:/Program Files/OpenNI2/Include/" CACHE STRING "..." FORCE)

STRING(REGEX REPLACE "Lib" "Redist" OPENNI2_REDIST "${OPENNI2_LIBRARY}")
STRING(REGEX REPLACE "/OpenNI2.lib" "" OPENNI2_REDIST "${OPENNI2_REDIST}")
MESSAGE(STATUS "OPENNI2_REDIST = ${OPENNI2_REDIST}")

find_package(OpenNI2 REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS} ${CUDA_INCLUDE_DIRS} ${OPENNI2_INCLUDE_DIRS})

# ---[ misc settings
if(USE_PROJECT_FOLDERS)
  set_property(GLOBAL PROPERTY USE_FOLDERS ON)
  set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMakeTargets")
endif()

# ---[ cuda settings
set(HAVE_CUDA 1)
#list(APPEND CUDA_NVCC_FLAGS "-arch;compute_20")
list(APPEND CUDA_NVCC_FLAGS "-gencode;arch=compute_20,code=sm_20;-gencode;arch=compute_20,code=sm_21;-gencode;arch=compute_30,code=sm_30;-gencode;arch=compute_35,code=sm_35")

if(UNIX OR APPLE)
  list(APPEND CUDA_NVCC_FLAGS "-Xcompiler;-fPIC;")
endif()

warnings_disable(CMAKE_CXX_FLAGS /wd4985)

add_subdirectory(kfusion)
add_subdirectory(apps)
