include_directories(${CMAKE_SOURCE_DIR}/kfusion/include)

file(GLOB srcs *.cpp *.hpp)
add_executable(demo ${srcs})
target_link_libraries(demo ${OpenCV_LIBS} kfusion)

set_target_properties(demo PROPERTIES
    DEBUG_POSTFIX "d"
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

install(TARGETS demo RUNTIME DESTINATION bin COMPONENT main)
install(FILES ${srcs} DESTINATION app COMPONENT main)

IF(EXISTS "${OpenCV_BIN_DIR_OPT}/opencv_core${OPENCV_VER}.dll")
	ADD_CUSTOM_COMMAND(TARGET demo POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${OpenCV_BIN_DIR_OPT}/opencv_core${OPENCV_VER}.dll" $<TARGET_FILE_DIR:demo>)
	ADD_CUSTOM_COMMAND(TARGET demo POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${OpenCV_BIN_DIR_OPT}/opencv_highgui${OPENCV_VER}.dll" $<TARGET_FILE_DIR:demo>)
	IF (OPENCV_VIZ_FOUND)
		IF(EXISTS "${OpenCV_BIN_DIR_OPT}/opencv_viz${OPENCV_VER}.dll")
			ADD_CUSTOM_COMMAND(TARGET demo POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${OpenCV_BIN_DIR_OPT}/opencv_viz${OPENCV_VER}.dll" $<TARGET_FILE_DIR:demo>)
		ELSE()
			MESSAGE(FATAL_ERROR "OpenCV viz module was found, but the libraries don't appear to have been built correctly. Corrupt modules will adversely affect the buld process. Please either reconfigure OpenCV with the viz module deselected, or re-build the viz module. The most common reason for failure to build the opencv viz module is linking to the incorrect version of VTK (e.g. 64-bit instead of 32-bit).")
		ENDIF()
	ENDIF()
ELSE()
	MESSAGE(WARNING "<${OpenCV_BIN_DIR_OPT}/opencv_core${OPENCV_VER}.dll> unable to be located. Has OpenCV been built in Release mode?")
ENDIF()

IF(EXISTS "${OpenCV_BIN_DIR_DBG}/opencv_core${OPENCV_VER}d.dll")
	ADD_CUSTOM_COMMAND(TARGET demo POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${OpenCV_BIN_DIR_DBG}/opencv_core${OPENCV_VER}d.dll" $<TARGET_FILE_DIR:demo>)
	ADD_CUSTOM_COMMAND(TARGET demo POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${OpenCV_BIN_DIR_DBG}/opencv_highgui${OPENCV_VER}d.dll" $<TARGET_FILE_DIR:demo>)
	IF (OPENCV_VIZ_FOUND)
		IF(EXISTS "${OpenCV_BIN_DIR_DBG}/opencv_viz${OPENCV_VER}d.dll")
			ADD_CUSTOM_COMMAND(TARGET demo POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different "${OpenCV_BIN_DIR_DBG}/opencv_viz${OPENCV_VER}d.dll" $<TARGET_FILE_DIR:demo>)
		ELSE()
			MESSAGE(FATAL_ERROR "OpenCV viz module was found, but the libraries don't appear to have been built correctly. Corrupt modules will adversely affect the buld process. Please either reconfigure OpenCV with the viz module deselected, or re-build the viz module. The most common reason for failure to build the opencv viz module is linking to the incorrect version of VTK (e.g. 64-bit instead of 32-bit).")
		ENDIF()
	ENDIF()
ELSE()
	MESSAGE(WARNING "<${OpenCV_BIN_DIR_OPT}/opencv_core${OPENCV_VER}d.dll> unable to be located. Has OpenCV been built in Debug mode?")
ENDIF()

IF(IS_DIRECTORY "${OPENNI2_REDIST}")
	ADD_CUSTOM_COMMAND(TARGET demo POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_directory "${OPENNI2_REDIST}" $<TARGET_FILE_DIR:demo>)
ENDIF()