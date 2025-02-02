cmake_minimum_required(VERSION 3.1)
project(corto)

option(BUILD_CORTO_CODEC_UNITY "Build the unity codec shared library of corto" ON)
option(BUILD_CORTO_EXE "Build the command line binary of corto" ON)

set (CMAKE_CXX_STANDARD 11)

SET(CORTO_SOURCE_PATH src)
SET(CORTO_HEADER_PATH include/corto)

SET(LIB_HEADERS
	${CORTO_HEADER_PATH}/bitstream.h
	${CORTO_HEADER_PATH}/color_attribute.h
	${CORTO_HEADER_PATH}/corto.h
	${CORTO_HEADER_PATH}/cstream.h
	${CORTO_HEADER_PATH}/decoder.h
	${CORTO_HEADER_PATH}/encoder.h
	${CORTO_HEADER_PATH}/index_attribute.h
	${CORTO_HEADER_PATH}/normal_attribute.h
	${CORTO_HEADER_PATH}/point.h
	${CORTO_HEADER_PATH}/tunstall.h
	${CORTO_HEADER_PATH}/vertex_attribute.h
	${CORTO_HEADER_PATH}/zpoint.h
	${CORTO_SOURCE_PATH}/corto_codec.h)

SET(LIB_SOURCES
	${CORTO_SOURCE_PATH}/bitstream.cpp
	${CORTO_SOURCE_PATH}/color_attribute.cpp
	${CORTO_SOURCE_PATH}/cstream.cpp
	${CORTO_SOURCE_PATH}/decoder.cpp
	${CORTO_SOURCE_PATH}/encoder.cpp
	${CORTO_SOURCE_PATH}/normal_attribute.cpp
	${CORTO_SOURCE_PATH}/tunstall.cpp
	${CORTO_SOURCE_PATH}/corto_codec.cpp)

SET(EXE_SOURCE
	${CORTO_SOURCE_PATH}/main.cpp
	${CORTO_SOURCE_PATH}/meshloader.cpp
	${CORTO_SOURCE_PATH}/tinyply.cpp)

SET(EXE_HEADERS
	${CORTO_SOURCE_PATH}/meshloader.h
	${CORTO_SOURCE_PATH}/tinyply.h
	${CORTO_SOURCE_PATH}/objload.h)

SET(CORTO_DEFINITIONS "")

if(MSVC)
	SET(CORTO_CFLAGS /nologo /W3)
	LIST(APPEND CORTO_DEFINITIONS _CRT_SECURE_NO_DEPRECATE)
else()
	SET(CORTO_CFLAGS -O2 -W -Wall -c -std=c++11 -pedantic)
endif()

ADD_LIBRARY(corto STATIC ${LIB_SOURCES} ${LIB_HEADERS})

SET(CORTOLIB_CFLAGS ${CORTO_CFLAGS})

if("${CMAKE_SYSTEM_NAME}" MATCHES "Linux")
	LIST(APPEND CORTOLIB_CFLAGS -fPIC)
elseif(APPLE)
	LIST(APPEND CORTOLIB_CFLAGS -fvisibility=hidden)
	if(IOS)
		LIST(APPEND CORTOLIB_CFLAGS -fembed-bitcode)
		set_xcode_property(corto IPHONEOS_DEPLOYMENT_TARGET "9.2")
	endif(IOS)
endif()

target_include_directories(corto PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_include_directories(corto PRIVATE ${CORTO_HEADER_PATH})

target_compile_definitions(corto PUBLIC ${CORTO_DEFINITIONS})
target_compile_options    (corto PUBLIC ${CORTOLIB_CFLAGS})
set_target_properties     (corto PROPERTIES DEBUG_POSTFIX "d")

INSTALL(TARGETS corto
	RUNTIME DESTINATION bin
	BUNDLE DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib)

INSTALL(FILES
	${CORTO_HEADER_PATH}/bitstream.h
	${CORTO_HEADER_PATH}/color_attribute.h
	${CORTO_HEADER_PATH}/corto.h
	${CORTO_HEADER_PATH}/cstream.h
	${CORTO_HEADER_PATH}/decoder.h
	${CORTO_HEADER_PATH}/encoder.h
	${CORTO_HEADER_PATH}/index_attribute.h
	${CORTO_HEADER_PATH}/normal_attribute.h
	${CORTO_HEADER_PATH}/point.h
	${CORTO_HEADER_PATH}/tunstall.h
	${CORTO_HEADER_PATH}/vertex_attribute.h
	${CORTO_HEADER_PATH}/zpoint.h
	DESTINATION include/corto)

if (BUILD_CORTO_CODEC_UNITY)
	ADD_LIBRARY(cortocodec_unity SHARED ${LIB_SOURCES} ${LIB_HEADERS})
	target_include_directories(cortocodec_unity PUBLIC ${CORTO_HEADER_PATH})
	if (${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
		# G++
		target_compile_options(cortocodec_unity PRIVATE -Wall -Wextra)
	elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "MSVC")
		# MSVC
		target_compile_options(cortocodec_unity PRIVATE /EHsc /MTd /W2 /c)
		# Set the DLLEXPORT variable to export symbols
		target_compile_definitions(cortocodec_unity PRIVATE WIN_EXPORT)
	endif()
endif()
if (BUILD_CORTO_EXE)
	ADD_EXECUTABLE(cortoexe ${EXE_SOURCE} ${EXE_HEADERS})
	SET(CORTOEXE_CFLAGS ${CORTO_CFLAGS})
	if(APPLE)
		if(IOS)
			set_xcode_property(cortoexe IPHONEOS_DEPLOYMENT_TARGET "9.2")
		endif()
	endif()
	target_include_directories(cortoexe PUBLIC ${CORTO_HEADER_PATH})
	target_link_libraries(cortoexe PUBLIC corto)

	target_compile_options    (cortoexe PUBLIC ${CORTOEXE_CFLAGS})
	set_target_properties     (cortoexe PROPERTIES OUTPUT_NAME "corto")

	INSTALL(TARGETS cortoexe
		RUNTIME DESTINATION bin
		BUNDLE DESTINATION bin
		LIBRARY DESTINATION lib
		ARCHIVE DESTINATION lib)
endif()
