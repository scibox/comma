PROJECT( "comma" )

cmake_minimum_required(VERSION 2.6)

ENABLE_TESTING()

# ---------------------------------------------------------------------------
# set compiler and linker flags and variables

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING
      "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
      FORCE)
ENDIF(NOT CMAKE_BUILD_TYPE)

IF(CMAKE_BUILD_TOOL MATCHES "(msdev|devenv|nmake|VCExpress)")
  SET(CMAKE_CXX_WARNING_LEVEL 4)
  IF(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
    STRING(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  ELSE(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
  ENDIF(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
  ADD_DEFINITIONS(/wd4512)
  ADD_DEFINITIONS(/wd4127)
  ADD_DEFINITIONS(/wd4996)
ENDIF(CMAKE_BUILD_TOOL MATCHES "(msdev|devenv|nmake|VCExpress)")

IF(CMAKE_BUILD_TOOL MATCHES "make")
  IF(NOT CMAKE_CXX_FLAGS MATCHES "-Wall")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
  ENDIF(NOT CMAKE_CXX_FLAGS MATCHES "-Wall")
  IF(NOT CMAKE_C_FLAGS MATCHES "-Wall")
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall")
  ENDIF(NOT CMAKE_C_FLAGS MATCHES "-Wall")
ENDIF(CMAKE_BUILD_TOOL MATCHES "make")

SET( SOURCE_CODE_BASE_DIR ${CMAKE_CURRENT_SOURCE_DIR} )
SET( LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib )

SET( EXECUTABLE_OUTPUT_PATH
     ${PROJECT_BINARY_DIR}/bin CACHE PATH
     "Single output directory for building all executables." )

INCLUDE_DIRECTORIES( ${SOURCE_CODE_BASE_DIR}/../ )

SET( comma_LIBRARY_PROPERTIES ${comma_LIBRARY_PROPERTIES} DEBUG_POSTFIX "-d" )

IF( NOT comma_INSTALL_BIN_DIR )
  SET( comma_INSTALL_BIN_DIR bin )
ENDIF( NOT comma_INSTALL_BIN_DIR )

IF( NOT comma_INSTALL_INCLUDE_DIR )
    SET( comma_INSTALL_INCLUDE_DIR include/${PROJECT_NAME} )
ENDIF( NOT comma_INSTALL_INCLUDE_DIR )

IF( NOT comma_INSTALL_LIB_DIR)
  SET( comma_INSTALL_LIB_DIR lib )
ENDIF( NOT comma_INSTALL_LIB_DIR )

IF( NOT comma_INSTALL_PACKAGE_DIR )
  SET( comma_INSTALL_PACKAGE_DIR CMakeFiles )
ENDIF( NOT comma_INSTALL_PACKAGE_DIR )

IF( APPLE )
    SET( CMAKE_INSTALL_NAME_DIR ${CMAKE_INSTALL_PREFIX}/${comma_INSTALL_LIB_DIR} )
ENDIF( APPLE )

OPTION( BUILD_TESTS "build unit tests" OFF )
SET( comma_BUILD_TESTS ${BUILD_TESTS} )

OPTION( BUILD_APPLICATIONS "build applications" OFF )
SET( comma_BUILD_APPLICATIONS ${BUILD_APPLICATIONS} )

OPTION( comma_BUILD_ZEROMQ "build io with zeromq support" OFF )

IF( WIN32 )
OPTION(BUILD_SHARED_LIBS "build with shared libraries" OFF)
ELSE( WIN32 )
OPTION(BUILD_SHARED_LIBS "build with shared libraries" ON)
ENDIF( WIN32 )
SET(comma_BUILD_SHARED_LIBS ${BUILD_SHARED_LIBS})

IF( NOT BUILD_SHARED_LIBS )
    SET( Boost_USE_STATIC_LIBS ON )
ENDIF( NOT BUILD_SHARED_LIBS )
FIND_PACKAGE( Boost COMPONENTS thread filesystem system date_time )
INCLUDE_DIRECTORIES( ${Boost_INCLUDE_DIRS} )
LINK_DIRECTORIES( ${Boost_LIBRARY_DIRS} )

IF( comma_BUILD_TESTS )
    #FIND_PACKAGE( GTest )
    #INCLUDE_DIRECTORIES(${GTEST_INCLUDE_DIRS})
    FIND_PATH( GTest_SOURCE_DIR NAMES CMakeLists.txt HINTS /usr/src/gtest DOC "source code of gtest" )
    ADD_SUBDIRECTORY( ${GTest_SOURCE_DIR} gtest )
    INCLUDE_DIRECTORIES( src ${GTest_SOURCE_DIR}/include ${GTest_SOURCE_DIR} )
    SET( GTEST_BOTH_LIBRARIES gtest gtest_main )
ENDIF( comma_BUILD_TESTS )

IF( comma_BUILD_ZEROMQ )
    SET( ZeroMQ_LIBRARY "zmq" )
    ADD_DEFINITIONS( -DUSE_ZEROMQ )
ENDIF( comma_BUILD_ZEROMQ )

SET( comma_ALL_LIBRARIES 
        debug comma_application-d
        optimized comma_application
        debug comma_base-d
        optimized comma_base
        debug comma_csv-d
        optimized comma_csv
        debug comma_io-d
        optimized comma_io
        debug comma_string-d
        optimized comma_string
        debug comma_xpath-d
        optimized comma_xpath )

IF(WIN32)
    SET( comma_ALL_EXTERNAL_LIBRARIES Ws2_32.lib )
ENDIF(WIN32)

IF(UNIX AND NOT QNXNTO AND NOT APPLE)
    SET( comma_ALL_EXTERNAL_LIBRARIES rt pthread )
ENDIF(UNIX AND NOT QNXNTO AND NOT APPLE)

SET( comma_ALL_EXTERNAL_LIBRARIES ${comma_ALL_EXTERNAL_LIBRARIES} ${Boost_LIBRARIES} )
SET( comma_ALL_EXTERNAL_LIBRARIES ${comma_ALL_EXTERNAL_LIBRARIES} )
IF( comma_BUILD_ZEROMQ )
    SET( comma_ALL_EXTERNAL_LIBRARIES ${comma_ALL_EXTERNAL_LIBRARIES} ${ZeroMQ_LIBRARY} )
ENDIF( comma_BUILD_ZEROMQ )

SET( comma_ALL_LIBRARIES ${comma_ALL_LIBRARIES} ${comma_ALL_LIBRARIES} ${comma_ALL_EXTERNAL_LIBRARIES} ) # as gcc parses library list only once
SET( comma_EXTERNAL_INCLUDES ${Boost_INCLUDE_DIRS} )

ADD_CUSTOM_TARGET( uninstall
    COMMAND ${CMAKE_COMMAND} -D CMAKE_INSTALL_MANIFEST="${CMAKE_CURRENT_BINARY_DIR}/install_manifest.txt" -P ${CMAKE_CURRENT_SOURCE_DIR}/CMakeFiles/cmake_uninstall.cmake )

FILE( GLOB comma_PACKAGE_CMAKEFILES ${CMAKE_CURRENT_SOURCE_DIR}/CMakeFiles/*.cmake )

INCLUDE( ${CMAKE_CURRENT_SOURCE_DIR}/CMakeFiles/generate_${PROJECT_NAME}_config.cmake )

CONFIGURE_FILE( ${PROJECT_SOURCE_DIR}/CMakeFiles/use_${PROJECT_NAME}.cmake.in
                ${PROJECT_BINARY_DIR}/CMakeFiles/use_${PROJECT_NAME}.cmake COPYONLY IMMEDIATE )

SET( comma_PACKAGE_CMAKEFILES ${comma_PACKAGE_CMAKEFILES} ${PROJECT_BINARY_DIR}/CMakeFiles/use_${PROJECT_NAME}.cmake )
INSTALL( FILES ${comma_PACKAGE_CMAKEFILES}
         DESTINATION ${CMAKE_INSTALL_PREFIX}/${comma_INSTALL_PACKAGE_DIR} )

#-----------------------------------------------------------------------------
# add sources

ADD_SUBDIRECTORY( application )
ADD_SUBDIRECTORY( base )
ADD_SUBDIRECTORY( containers )
ADD_SUBDIRECTORY( csv )
ADD_SUBDIRECTORY( dispatch )
ADD_SUBDIRECTORY( io )
ADD_SUBDIRECTORY( math )
ADD_SUBDIRECTORY( name_value )
ADD_SUBDIRECTORY( packed )
ADD_SUBDIRECTORY( string )
ADD_SUBDIRECTORY( sync )
ADD_SUBDIRECTORY( visiting )
ADD_SUBDIRECTORY( xpath )
