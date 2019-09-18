cmake_minimum_required(VERSION 3.0)


project(schnapps-superbuild)

#### Here are located the FindPackages that we need
#list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules")
list(APPEND CMAKE_PREFIX_PATH "${CMAKE_CURRENT_BINARY_DIR}")

include(ExternalProject)

set(installDir ${CMAKE_CURRENT_BINARY_DIR}/install)
set(sourceDir ${CMAKE_CURRENT_SOURCE_DIR})
set(qt5Dir ${Qt5_DIR})

set(toolChain ${CMAKE_TOOLCHAIN_FILE})

ExternalProject_Add(cgogn
  SOURCE_DIR ${sourceDir}/cgogn
  BINARY_DIR ${CMAKE_BINARY_DIR}/cgogn-build
  INSTALL_DIR ${installDir}

  GIT_REPOSITORY https://github.com/cgogn/CGoGN_2.git
  GIT_REMOTE_NAME origin
  GIT_TAG develop
  CMAKE_ARGS
    -DCMAKE_BUILD_TYPE:STRING=${BUILD_TYPE}
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -DCMAKE_PREFIX_PATH:PATH=<INSTALL_DIR>
    -DCMAKE_TOOLCHAIN_FILE:PATH=${toolChain}
    -DQt5_DIR:PATH=${qt5Dir}
    -DCGOGN_MODULE_topology:BOOL=OFF
)

ExternalProject_Add(schnapps
  SOURCE_DIR ${sourceDir}/schnapps
  BINARY_DIR ${CMAKE_BINARY_DIR}/schnapps-build
  INSTALL_DIR ${installDir}

  GIT_REPOSITORY https://github.com/cgogn/SCHNApps.git
  GIT_REMOTE_NAME origin
  GIT_TAG develop
  CMAKE_ARGS
    -DCMAKE_BUILD_TYPE:STRING=${BUILD_TYPE}
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -DCMAKE_PREFIX_PATH:PATH=<INSTALL_DIR>
    -DCMAKE_TOOLCHAIN_FILE:PATH=${toolChain}
    -DQt5_DIR:PATH=${qt5Dir}
   	-DUSE_PLUGIN_attribute_editor:BOOL=ON
   	-DUSE_PLUGIN_cage_3d_deformation:BOOL=OFF
   	-DUSE_PLUGIN_cmap_provider:BOOL=ON
   	-DUSE_PLUGIN_export:BOOL=ON
  	-DUSE_PLUGIN_extract_surface:BOOL=OFF
  	-DUSE_PLUGIN_image:BOOL=OFF
  	-DUSE_PLUGIN_import:BOOL=ON
  	-DUSE_PLUGIN_merge:BOOL=OFF
  	-DUSE_PLUGIN_meshgen:BOOL=OFF
    -DUSE_PLUGIN_point_set_render:BOOL=ON
    -DUSE_PLUGIN_polyline_render:BOOL=ON
    -DUSE_PLUGIN_render_transparency:BOOL=ON
  	-DUSE_PLUGIN_surface_deformation:BOOL=ON
  	-DUSE_PLUGIN_surface_differential_properties:BOOL=ON
  	-DUSE_PLUGIN_surface_modelisation:BOOL=OFF
  	-DUSE_PLUGIN_surface_render:BOOL=ON
  	-DUSE_PLUGIN_surface_render_scalar:BOOL=ON
  	-DUSE_PLUGIN_surface_render_vector:BOOL=ON
    -DUSE_PLUGIN_surface_selection:BOOL=ON
  	-DUSE_PLUGIN_volume_render:BOOL=OFF
  DEPENDS cgogn
)


ExternalProject_Add_StepDependencies(schnapps install cgogn)


set(CPACK_INSTALL_CMAKE_PROJECTS "${CPACK_INSTALL_CMAKE_PROJECTS};${CMAKE_BINARY_DIR}/cgogn-build;cgogn;ALL;/")
set(CPACK_INSTALL_CMAKE_PROJECTS "${CPACK_INSTALL_CMAKE_PROJECTS};${CMAKE_BINARY_DIR}/schnapps-build;schnapps;ALL;/")
set(CPACK_GENERATOR "WIX")


SET(CPACK_INSTALL_DIRECTORY "$PROGRAMFILES64")

set(CPACK_PACKAGE_CONTACT "Lionel Untereiner <untereiner@geosiris.com>" )
set(CPACK_PACKAGE_NAME "SCHNApps")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "SCHNApps")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "A geometric modeling tool based on combinatorial maps")
set(CPACK_PACKAGE_VENDOR "geosiris.com")
set(CPACK_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}-installer")
set(CPACK_PACKAGE_EXECUTABLES "schnapps;SCHNApps")

set(CPACK_VERBATIM_VARIABLES YES)

# set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
# set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
# set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
# 

#set(CPACK_PACKAGE_DESCRIPTION_FILE
#set(CPACK_RESOURCE_FILE_WELCOME
#set(CPACK_RESOURCE_FILE_LICENSE
#set(CPACK_RESOURCE_FILE_README


set(CPACK_WIX_PROGRAM_MENU_FOLDER "SCHNApps")
set(CPACK_WIX_UPGRADE_GUID c3116708-b4f9-11e8-91c9-b3d926a9af6d)
set(CPACK_WIX_PRODUCT_GUID 2a18d152-b4fa-11e8-9d42-db9a783d7046)

#Add extra install files
#list(APPEND CPACK_WIX_EXTRA_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/extra/wix/SCHNApps_InstallDirDlg.wxs")


include(CPack)

