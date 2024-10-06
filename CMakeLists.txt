cmake_minimum_required(VERSION 3.10)
project(MyOFXPlugin)

# Set the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Directory where the OpenFX header files are located dfvdg
# Adjust this path according to the location of your OpenFX files
set(OFX_INCLUDE_DIR "C:/OpenFX14/include")

# Add the include directory
include_directories(${OFX_INCLUDE_DIR})

# Add your plugin's source files ff
# You can list the files manually or use a glob expression
file(GLOB PLUGIN_SOURCES
    src/MyPlugin.cpp
)

# Create the shared library (the plugin)
add_library(MyOFXPlugin MODULE ${PLUGIN_SOURCES})

# On UNIX systems, it's necessary to add the .ofx suffix
set_target_properties(MyOFXPlugin PROPERTIES
    PREFIX ""
    SUFFIX ".ofx"
)

# Specific configurations for macOS
if(APPLE)
    set_target_properties(MyOFXPlugin PROPERTIES
        BUNDLE TRUE
        BUNDLE_EXTENSION "ofx"
    )
endif()

# Optional: Copy the plugin to the OpenFX plugin folder after building
# Adjust OFX_PLUGIN_INSTALL_DIR according to your system

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

install(TARGETS MyOFXPlugin
    LIBRARY DESTINATION ${OFX_PLUGIN_INSTALL_DIR}
    RUNTIME DESTINATION ${OFX_PLUGIN_INSTALL_DIR}
    BUNDLE  DESTINATION ${OFX_PLUGIN_INSTALL_DIR}
)

