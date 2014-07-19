set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "--ftz=true;--prec-div=false;--prec-sqrt=false")
add_module_library(kfusion)
target_link_libraries(kfusion ${CUDA_LIBRARIES} ${CUDA_CUDA_LIBRARY} ${OpenCV_LIBS} ${OPENNI2_LIBRARIES})
set_target_properties (kfusion PROPERTIES COMPILE_FLAGS ${GENERIC_COMPILE_FLAGS})

IF(IS_DIRECTORY "${OPENNI2_REDIST}")
	ADD_CUSTOM_COMMAND(TARGET kfusion POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_directory "${OPENNI2_REDIST}" $<TARGET_FILE_DIR:kfusion>)
ENDIF()