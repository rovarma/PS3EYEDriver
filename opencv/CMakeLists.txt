cmake_minimum_required(VERSION 3.0)
project(ps3eye_opencv VERSION 0.1.0)

set(PSEYE_OPENCV_INCL_DIRS)
set(PSEYE_OPENCV_REQ_LIBS)

LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}")
IF (NOT MSVC)
    set(CMAKE_CXX_FLAGS "-std=c++11 -stdlib=libc++")
ENDIF()

# Platform specific libraries
IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    find_library(IOKIT_FRAMEWORK IOKit)
    find_library(COREFOUNDATION_FRAMEWORK CoreFoundation)
    #find_library(QUARTZCORE QuartzCore)
    find_library(APPKIT_FRAMEWORK AppKit)
    #find_library(QTKIT QTKit)
    find_library(AVFOUNDATION AVFoundation)
    #find_library(IOBLUETOOTH IOBluetooth)
    #stdc++ ${QUARTZCORE} ${APPKIT_FRAMEWORK} ${QTKIT} ${AVFOUNDATION}
    list(APPEND PSEYE_OPENCV_REQ_LIBS
        ${COREFOUNDATION_FRAMEWORK}
        ${IOKIT_FRAMEWORK}
        ${APPKIT_FRAMEWORK}
        ${AVFOUNDATION})
ENDIF()

# ps3eye
list(APPEND PSEYE_OPENCV_INCL_DIRS ${CMAKE_CURRENT_LIST_DIR}/../src)

# OpenCV
# Override by adding "-DOpenCV_DIR=C:\path\to\opencv\build" to your cmake command
IF(NOT OpenCV_DIR)
	IF(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
		# Default location of pre-compiled OpenCV for Windows
		set(OpenCV_DIR "C:/OpenCV-3.0.0/build")# CACHE PATH "Path to OpenCV" FORCE)
	ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
		# Location of homebrew opencv3
		# Alternatively, can do `brew link opencv3 --force`
		message(STATUS "Using homebrew opencv3")
		set(OpenCV_DIR "/usr/local/Cellar/opencv3/3.1.0_3/share/OpenCV")
	ENDIF()#Windows or Darwin
ENDIF(NOT OpenCV_DIR)
set(OpenCV_STATIC ON)
FIND_PACKAGE(OpenCV REQUIRED)
IF ("${OpenCV_LIBS}" STREQUAL "")
	message("OpenCV_LIBS was empty! Manually setting.")
	list(APPEND OpenCV_LIBS opencv_core opencv_calib3d opencv_highgui)
ENDIF()
IF ("${OpenCV_INCLUDE_DIRS}" STREQUAL "" AND ${OpenCV_DIR})
	message("OpenCV_INCLUDE_DIRS was empty! Manually setting.")
	set (OpenCV_INCLUDE_DIRS ${OpenCV_DIR}\include)
ENDIF()
list(APPEND PSEYE_OPENCV_INCL_DIRS ${OpenCV_INCLUDE_DIRS})
list(APPEND PSEYE_OPENCV_REQ_LIBS ${OpenCV_LIBS})

# LibuSUSB
find_package(USB1 REQUIRED)
list(APPEND PSEYE_OPENCV_INCL_DIRS ${LIBUSB_INCLUDE_DIR})
list(APPEND PSEYE_OPENCV_REQ_LIBS ${LIBUSB_LIBRARIES})

add_executable(ps3eye_opencv
	${CMAKE_CURRENT_LIST_DIR}/../src/ps3eye.cpp
	${CMAKE_CURRENT_LIST_DIR}/main.cpp)
target_include_directories(ps3eye_opencv PUBLIC
	${PSEYE_OPENCV_INCL_DIRS})
target_link_libraries(ps3eye_opencv
	${PSEYE_OPENCV_REQ_LIBS})