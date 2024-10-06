cmake_minimum_required(VERSION 3.10)
project(MyOFXPlugin)

# Set the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Directory where the OpenFX header files are located dfvdg
# Adjust this path according to the location of your OpenFX files

file(GLOB OFX_LIB_HEADERS
    OpenFX14/Support/include/*.h
    OpenFX14/include/*.h
)

set(OFX_LIB_SOURCES
    OpenFX14/Support/Library/ofxsCore.cpp
    OpenFX14/Support/Library/ofxsImageEffect.cpp
    OpenFX14/Support/Library/ofxsInteract.cpp
    OpenFX14/Support/Library/ofxsLog.cpp
    OpenFX14/Support/Library/ofxsMultiThread.cpp
    OpenFX14/Support/Library/ofxsParams.cpp
    OpenFX14/Support/Library/ofxsProperty.cpp
    OpenFX14/Support/Library/ofxsPropertyValidation.cpp
)

add_library(OFX_LIB STATIC ${OFX_LIB_SOURCES} ${OFX_LIB_HEADERS})
target_include_directories(OFX_LIB PUBLIC
    OpenFX14/Support/include/
    OpenFX14/include/
)

file(GLOB PLUGIN_SOURCES
    src/MyPlugin.cpp
)

# Add executable
add_library(${CMAKE_PROJECT_NAME} SHARED ${PLUGIN_SOURCES} ${OFX_LIB_HEADERS})
target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC
    OpenFX14/Support/include/
    OpenFX14/include/
)

if(APPLE)
  set(OFX_PLUGIN_INSTALL_DIR "/Library/OFX/Plugins")
  set(ARCHDIR "MacOS")
elseif(WIN32)
  set(OFX_PLUGIN_INSTALL_DIR "C:/Program Files/Common Files/OFX/Plugins")
  set(ARCHDIR "Win64")
elseif(UNIX)
  set(OFX_PLUGIN_INSTALL_DIR "/usr/OFX/Plugins")
  set(ARCHDIR "Linux-x86-64")
else()
  set(OFX_PLUGIN_INSTALL_DIR "/unknown-os")
  set(ARCHDIR "unknown-arch")
endif()

option(AUTO_DEPLOY "Install the plugin on build" ON)

if(AUTO_DEPLOY)
set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES SUFFIX ".ofx" PREFIX "")
if (DEFINED ENV{OFX_PLUGIN_PATH})
# use the standard envvar OFX_PLUGIN_PATH (custom plugin location) this could avoid write permissions problems
# set(OFX_PLUGIN_PATH $ENV{OFX_PLUGIN_PATH})
# avoid backslashes under windows...
file(TO_CMAKE_PATH $ENV{OFX_PLUGIN_PATH} OFX_PLUGIN_PATH)
message( "The ofx plugin will be installed in OFX_PLUGIN_PATH : ${OFX_PLUGIN_PATH}/${CMAKE_PROJECT_NAME}.ofx.bundle/Contents/${ARCHDIR}")
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND}  -E copy_if_different $<TARGET_FILE:${CMAKE_PROJECT_NAME}>
                                    ${OFX_PLUGIN_PATH}/${CMAKE_PROJECT_NAME}.ofx.bundle/Contents/${ARCHDIR}/$<TARGET_FILE_NAME:${CMAKE_PROJECT_NAME}>
)
# install(TARGETS ${TARGET} DESTINATION "${OFX_PLUGIN_PATH}/${TARGET}.ofx.bundle/Contents/${ARCHDIR}")
else()
# use the PLUGINDIR (standard plugin location)
message( "The OFX_PLUGIN_PATH env var is not defined : the ofx plugin will be installed in standard location : ${PLUGINDIR}/${CMAKE_PROJECT_NAME}.ofx.bundle/Contents/${ARCHDIR}")
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND}  -E copy_if_different $<TARGET_FILE:${CMAKE_PROJECT_NAME}>
                                    ${PLUGINDIR}/${CMAKE_PROJECT_NAME}.ofx.bundle/Contents/${ARCHDIR}/$<TARGET_FILE_NAME:${CMAKE_PROJECT_NAME}>
)
# install(TARGETS ${TARGET} DESTINATION "${PLUGINDIR}/${TARGET}.ofx.bundle/Contents/${ARCHDIR}")
endif()
endif()
