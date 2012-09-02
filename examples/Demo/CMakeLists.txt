# TIXI Library
# @author Martin Siggel
# 
# The demo serves for 2 purposes. First,k it demonstrated the usage of tixi.
# Second, it allows us to scan for 3rd party library dependencies and install
# them with tixi (cmake doesn't allow scanning dlls only).

set(DEMO_SRC tixiDemo.c)

include_directories(${PROJECT_SOURCE_DIR}/src)

if(CMAKE_COMPILER_IS_GNUCC)
	set(CMAKE_C_FLAGS "-Wall -fmessage-length=0")
endif()

add_executable(tixiDemo ${DEMO_SRC})
target_link_libraries (tixiDemo TIXI)

add_custom_command(
	TARGET tixiDemo
	POST_BUILD
	COMMAND ${CMAKE_COMMAND}
	ARGS -E copy ${CMAKE_CURRENT_SOURCE_DIR}/howtoin.xml ${CMAKE_CURRENT_BINARY_DIR}/howtoin.xml
)
#--------------------------------------------------------------------------------
# Now the installation stuff below
#--------------------------------------------------------------------------------

SET(APPS "\${CMAKE_INSTALL_PREFIX}/bin/tixiDemo")
IF(APPLE)
  SET(APPS "\${CMAKE_INSTALL_PREFIX}/tixiDemo.app")
ENDIF(APPLE)
IF(WIN32)
  SET(APPS "\${CMAKE_INSTALL_PREFIX}/bin/tixiDemo.exe")
ENDIF(WIN32)

#--------------------------------------------------------------------------------
# Install the tixiDemo application, on Apple, the bundle is at the root of the
# install tree, and on other platforms it'll go into the bin directory.
INSTALL(TARGETS tixiDemo 
    BUNDLE DESTINATION . COMPONENT Runtime
    RUNTIME DESTINATION bin COMPONENT Runtime
    )

# directories to look for dependencies
SET(DIRS ${ADD_LIB_PATH} ${XML_INCLUDE_DIR} ${XSLT_INCLUDE_DIR} ${CURL_INCLUDE_DIR})

if (WIN32 OR APPLE)

# Now the work of copying dependencies into the bundle/package
# The quotes are escaped and variables to use at install time have their $ escaped
# An alternative is the do a configure_file() on a script and use install(SCRIPT  ...).
INSTALL(CODE "
    include(BundleUtilities)
    fixup_bundle(\"${APPS}\" \"\" \"${DIRS}\")
    " COMPONENT Runtime)

endif()