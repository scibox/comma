SET( PROJECT "name_value" )
SET( TARGET_NAME comma_${PROJECT} )

FILE( GLOB source   ${SOURCE_CODE_BASE_DIR}/${PROJECT}/*.cpp)
FILE( GLOB includes ${SOURCE_CODE_BASE_DIR}/${PROJECT}/*.h )
FILE( GLOB impl_includes ${SOURCE_CODE_BASE_DIR}/${PROJECT}/impl/*.h )
SOURCE_GROUP( ${PROJECT} FILES ${source} ${includes} ${impl_includes} )
ADD_CUSTOM_TARGET( ${TARGET_NAME} ${source} ${includes} ${impl_includes} ${comma_EXTERNAL_ALL_LIBRARIES} comma_application comma_string comma_xpath )

INSTALL( FILES ${includes} DESTINATION ${CMAKE_INSTALL_PREFIX}/${comma_INSTALL_INCLUDE_DIR}/${PROJECT} )
INSTALL( FILES ${impl_includes} DESTINATION ${CMAKE_INSTALL_PREFIX}/${comma_INSTALL_INCLUDE_DIR}/${PROJECT}/impl )

IF( comma_BUILD_APPLICATIONS )
    ADD_SUBDIRECTORY( applications )
ENDIF( comma_BUILD_APPLICATIONS )

IF( comma_BUILD_TESTS )
    ADD_SUBDIRECTORY( test )
ENDIF( comma_BUILD_TESTS )