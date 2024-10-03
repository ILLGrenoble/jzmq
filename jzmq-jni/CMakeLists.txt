# CMake build script for ØMQ Java bindings on Windows

cmake_minimum_required (VERSION 3.20)

project (JZMQ)
find_package (Java REQUIRED)
find_package (JNI REQUIRED)
find_package (ZeroMQ QUIET)

if(NOT ZeroMQ_FOUND)
  message(STATUS "ZeroMQ was not found by package")
endif()

set(ZMQ_C_INCLUDE_PATH "" CACHE PATH "Path to ZMQ Header files")
set(ZMQ_C_LIB_PATH "" CACHE PATH "Path to ZMQ library")

# Search for javac which replaces javah
find_program (JNI_JAVAC
  NAMES javac
	HINTS ${_JAVA_HINTS}
	PATHS ${_JAVA_PATHS}
)

#-----------------------------------------------------------------------------
# force off-tree build


if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
message(FATAL_ERROR "CMake generation is not allowed within the source directory!
Remove the CMakeCache.txt file and try again from another folder, e.g.:
   del CMakeCache.txt
   mkdir build
   cd build
   cmake ..
")
endif(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})

#-----------------------------------------------------------------------------
# default to Release build

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING
      "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
      FORCE)
endif(NOT CMAKE_BUILD_TYPE)

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH  ${CMAKE_BINARY_DIR}/lib)

#-----------------------------------------------------------------------------
# platform specifics

add_definitions(
        -DWIN32
	-DDLL_EXPORT
	-DFD_SETSIZE=1024
)

#-----------------------------------------------------------------------------
# source files

set(java-sources
	org/zeromq/ZMQ.java
	org/zeromq/ZMQException.java
	../../../../jzmq-devices/src/main/java/org/zeromq/ZMQForwarder.java
	../../../../jzmq-devices/src/main/java/org/zeromq/ZMQQueue.java
	../../../../jzmq-devices/src/main/java/org/zeromq/ZMQStreamer.java
	org/zeromq/EmbeddedLibraryTools.java
	org/zeromq/App.java
	../../../../jzmq-core/src/main/java/org/zeromq/ZContext.java
	../../../../jzmq-core/src/main/java/org/zeromq/Utils.java
	../../../../jzmq-core/src/main/java/org/zeromq/ZDispatcher.java
	../../../../jzmq-core/src/main/java/org/zeromq/ZFrame.java
	../../../../jzmq-core/src/main/java/org/zeromq/ZMsg.java
	../../../../jzmq-core/src/main/java/org/zeromq/ZLoop.java
	../../../../jzmq-core/src/main/java/org/zeromq/ZThread.java
)
set(java-classes
	org/zeromq/ZMQ.class
	org/zeromq/Utils.class
	org/zeromq/ZMQ$Context.class
	org/zeromq/ZMQ$Curve$KeyPair.class
	org/zeromq/ZMQ$Curve.class
	org/zeromq/ZMQ$Socket.class
	org/zeromq/ZMQ$PollItem.class
	org/zeromq/ZMQ$Poller.class
	org/zeromq/ZMQ$Error.class
	org/zeromq/ZMQ$Event.class
	org/zeromq/ZMQException.class
	org/zeromq/ZMQQueue.class
	org/zeromq/ZMQForwarder.class
	org/zeromq/ZMQStreamer.class
	org/zeromq/EmbeddedLibraryTools.class
	org/zeromq/App.class
	org/zeromq/ZContext.class
	org/zeromq/ZDispatcher.class
	org/zeromq/ZDispatcher$SocketDispatcher$1.class
	org/zeromq/ZDispatcher$SocketDispatcher$2.class
	org/zeromq/ZDispatcher$SocketDispatcher$ZMessageBuffer.class
	org/zeromq/ZDispatcher$SocketDispatcher.class
	org/zeromq/ZDispatcher$ZMessageHandler.class
	org/zeromq/ZDispatcher$ZSender.class
	org/zeromq/ZFrame.class
	org/zeromq/ZMsg.class
	org/zeromq/ZLoop.class
	org/zeromq/ZLoop$IZLoopHandler.class
	org/zeromq/ZLoop$SPoller.class
	org/zeromq/ZLoop$STimer.class
	org/zeromq/ZThread.class
	org/zeromq/ZThread$IAttachedRunnable.class
	org/zeromq/ZThread$IDetachedRunnable.class
	org/zeromq/ZThread$ShimThread.class
)
set(javah-headers
	org_zeromq_ZMQ.h
	org_zeromq_ZMQ_Error.h
	org_zeromq_ZMQ_Context.h
	org_zeromq_ZMQ_Socket.h
	org_zeromq_ZMQ_PollItem.h
	org_zeromq_ZMQ_Poller.h	
)
set(cxx-sources
	Context.cpp
	Poller.cpp
	Socket.cpp
	util.cpp
	ZMQ.cpp
)

include_directories(
        src
        src/main/java
	    ${CMAKE_CURRENT_BINARY_DIR}
)

#-----------------------------------------------------------------------------
# optional modules

add_definitions(
	-DZMQ_HAVE_OPENPGM
)
include_directories(
	${ZMQ_C_INCLUDE_PATH}
	${JNI_INCLUDE_DIRS}
)
link_directories(
	${ZMQ_C_LIB_PATH}
)

#-----------------------------------------------------------------------------
# source generators

foreach (source ${cxx-sources})
	list(APPEND sources ${CMAKE_CURRENT_SOURCE_DIR}/src/main/c++/${source})	
endforeach()

add_custom_command(
	OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/config.hpp
	COMMAND ${CMAKE_COMMAND}
	ARGS	-E
		copy
		${CMAKE_CURRENT_SOURCE_DIR}/builds/msvc/config.hpp
		${CMAKE_CURRENT_BINARY_DIR}/config.hpp
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/builds/msvc/config.hpp
)
list(APPEND sources ${CMAKE_CURRENT_BINARY_DIR}/config.hpp)

add_custom_command(
	OUTPUT ${javah-headers}
	COMMAND ${JNI_JAVAC}
	ARGS ${CMAKE_CURRENT_SOURCE_DIR}/src/main/java/org/zeromq/ZMQ.java
	     -cp ${CMAKE_CURRENT_BINARY_DIR}
	     -h ${CMAKE_CURRENT_BINARY_DIR}
	WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
	DEPENDS ${java-classes}
)
list(APPEND sources ${javah-headers})

set (source-tmp "")
foreach (source ${java-sources})
	list (APPEND source-tmp ${CMAKE_CURRENT_SOURCE_DIR}/src/main/java/${source})
endforeach()
set (java-sources ${source-tmp})

add_custom_command(
	OUTPUT ${java-classes}
	COMMAND ${JAVA_COMPILE}
	ARGS	-classpath ${CMAKE_CURRENT_BINARY_DIR}
		-sourcepath ${CMAKE_CURRENT_SOURCE_DIR}/src/main/java
		-d ${CMAKE_CURRENT_BINARY_DIR}
		${java-sources}
	DEPENDS ${java-sources}
)

add_custom_command(
	OUTPUT lib/zmq.jar
	COMMAND ${JAVA_ARCHIVE}
	ARGS	cf
		lib/zmq.jar
		${java-classes}
	WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
	DEPENDS ${java-classes}
)
list(APPEND sources lib/zmq.jar)

#-----------------------------------------------------------------------------
# output

add_library(jzmq SHARED ${sources})

if(ZeroMQ_FOUND)
  target_link_libraries(jzmq PRIVATE libzmq)
else()
  target_link_libraries(jzmq libzmq.lib)
endif()


set(docs
    AUTHORS
	COPYING
	COPYING.LESSER
	ChangeLog
	INSTALL
	NEWS
	README
	README-PERF
)

install (TARGETS jzmq DESTINATION lib)
install (FILES ${CMAKE_CURRENT_BINARY_DIR}/lib/zmq.jar DESTINATION lib)
install (FILES ${docs} DESTINATION doc)

# By default, do not warn when built on machines using only VS Express:
IF(NOT DEFINED CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS_NO_WARNINGS)
	SET(CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS_NO_WARNINGS ON)
ENDIF()

include (InstallRequiredSystemLibraries)
set (CPACK_PACKAGE_VENDOR "Miru Limited")
set (CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/COPYING")
set (CPACK_PACKAGE_VERSION_MAJOR "3")
set (CPACK_PACKAGE_VERSION_MINOR "2")
set (CPACK_PACKAGE_VERSION_PATCH "2")
include (CPack)

# end of file
