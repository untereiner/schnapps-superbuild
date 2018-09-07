cmake_minimum_required(VERSION 3.0)


function(dict command dict )
    if(command STREQUAL SET)
        set(arg_key ${ARGV2})
        set(arg_value ${ARGV3})

        dict(_IDX ${dict} "${arg_key}" idx)
        if(NOT idx STREQUAL -1)
            list(REMOVE_AT ${dict} ${idx})
        endif()

        list(APPEND ${dict} "${arg_key}=${arg_value}")
        set(${dict} "${${dict}}" PARENT_SCOPE)

    elseif(command STREQUAL GET)
        set(arg_key ${ARGV2})
        set(arg_outvar ${ARGV3})

        dict(_IDX ${dict} "${arg_key}" idx)
        if(idx STREQUAL -1)
            message(FATAL_ERROR "No key \"${arg_key}\" in dictionary")
        endif()

        list(GET ${dict} ${idx} kv)
        string(REGEX REPLACE "^[^=]+=(.*)" "\\1" value "${kv}")
        set(${arg_outvar} "${value}" PARENT_SCOPE)

    elseif(command STREQUAL _IDX)
        set(arg_key ${ARGV2})
        set(arg_outvar ${ARGV3})
        set(idx 0)
        foreach(kv IN LISTS ${dict})
            string(REGEX REPLACE "^([^=]+)=.*" "\\1" key "${kv}")
            if(arg_key STREQUAL key)
                set(${arg_outvar} "${idx}" PARENT_SCOPE)
                return()
            endif()
            math(EXPR idx ${idx}+1)
        endforeach()
        set(${arg_outvar} "-1" PARENT_SCOPE)

    else()
        message(FATAL_ERROR "dict does not recognize sub-command ${command}")
    endif()
endfunction()

function(get_cached_variables mydict)
	get_cmake_property(CACHE_VARS CACHE_VARIABLES)
	foreach(CACHE_VAR ${CACHE_VARS})
	  get_property(CACHE_VAR_HELPSTRING CACHE ${CACHE_VAR} PROPERTY HELPSTRING)
	  if(CACHE_VAR_HELPSTRING STREQUAL "No help, variable specified on the command line.")
	    #get_property(CACHE_VAR_TYPE CACHE ${CACHE_VAR} PROPERTY TYPE)
	    # if(CACHE_VAR_TYPE STREQUAL "UNINITIALIZED")
	    #   set(CACHE_VAR_TYPE)
	    # else()
	    #   set(CACHE_VAR_TYPE :${CACHE_VAR_TYPE})
	    # endif()
	    #list(APPEND values "${${CACHE_VAR}}")
	    #list(APPEND keys "${CACHE_VAR}") #"${CACHE_VAR}${CACHE_VAR_TYPE}=\"${${CACHE_VAR}}\"")
	    dict(SET mydict "${CACHE_VAR}" "${${CACHE_VAR}}" )
	  endif()
	endforeach()
	set(${mydict} PARENT_SCOPE)
endfunction()



get_cached_variables(mydict)
dict(GET mydict "Qt5_DIR" qt5_dir)

project(schnapps-superbuild)

#### Here are located the FindPackages that we need
#list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules")
list(APPEND CMAKE_PREFIX_PATH "${CMAKE_CURRENT_BINARY_DIR}")

include(ExternalProject)

set(installDir ${CMAKE_CURRENT_BINARY_DIR}/install)
set(sourceDir ${CMAKE_CURRENT_SOURCE_DIR})

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
    -DQt5_DIR:PATH=${qt5_dir}
    #-DBUILD_SHARED_LIBS:BOOL=ON 
)

ExternalProject_Add(schnapps
  SOURCE_DIR ${sourceDir}/schnapps
  BINARY_DIR ${CMAKE_BINARY_DIR}/schnapps-build
  INSTALL_DIR ${installDir}

  GIT_REPOSITORY https://github.com/untereiner/SCHNApps.git
  GIT_REMOTE_NAME origin
  GIT_TAG cmake
  #URL /Users/untereiner/Documents/Developments/cgogn2/SCHNApps/
  CMAKE_ARGS
    -DCMAKE_BUILD_TYPE:STRING=${BUILD_TYPE}
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -DCMAKE_PREFIX_PATH:PATH=<INSTALL_DIR>
    #-DCMAKE_INSTALL_NAME_DIR:STRING=@executable_path/lib/
    -DQt5_DIR:PATH=${qt5_dir}
 	-DUSE_PLUGIN_attribute_editor:BOOL=OFF
 	-DUSE_PLUGIN_cage_3d_deformation:BOOL=OFF
 	-DUSE_PLUGIN_empty_plugin:BOOL=ON
 	-DUSE_PLUGIN_export:BOOL=ON
	-DUSE_PLUGIN_extract_surface:BOOL=OFF
	-DUSE_PLUGIN_image:BOOL=OFF
	-DUSE_PLUGIN_import:BOOL=ON
	-DUSE_PLUGIN_merge:BOOL=OFF
	-DUSE_PLUGIN_meshgen:BOOL=OFF
	-DUSE_PLUGIN_selection:BOOL=ON  
	-DUSE_PLUGIN_shallow_water:BOOL=OFF
	-DUSE_PLUGIN_shallow_water_2:BOOL=OFF
	-DUSE_PLUGIN_surface_deformation:BOOL=OFF
	-DUSE_PLUGIN_surface_differential_properties:BOOL=OFF
	-DUSE_PLUGIN_surface_modelisation:BOOL=OFF
	-DUSE_PLUGIN_surface_render:BOOL=OFF
	-DUSE_PLUGIN_surface_render_scalar:BOOL=OFF
	-DUSE_PLUGIN_surface_render_transp:BOOL=OFF
	-DUSE_PLUGIN_surface_render_vector:BOOL=OFF
	-DUSE_PLUGIN_volume_modelisation:BOOL=OFF
	-DUSE_PLUGIN_volume_render:BOOL=OFF
    #-DBUILD_SHARED_LIBS:BOOL=ON 
    #-DUSE_PLUGIN_area_of_interest:BOOL=OFF
 	#-DUSE_PLUGIN_data_preprocessing:BOOL=OFF
)


ExternalProject_Add_StepDependencies(schnapps install cgogn)



	# find_package(Qt5Core REQUIRED)
	# get_target_property(_qmake_executable Qt5::qmake IMPORTED_LOCATION)
	# get_filename_component(_qt_bin_dir "${_qmake_executable}" DIRECTORY)
	# find_program(MACDEPLOYQT_EXECUTABLE macdeployqt HINTS "${_qt_bin_dir}")
	# if(APPLE AND NOT MACDEPLOYQT_EXECUTABLE)
	#     message(FATAL_ERROR "macdeployqt not found")
	# else()
	# 	message(STATUS "${MACDEPLOYQT_EXECUTABLE}")
	# endif()

    
 #    ExternalProject_Add_Step(schnapps post_build 
 #    	COMMENT "Deploying Qt... "
 #    	DEPENDEES build 
 #    	#DEPENDERS install
 #    	COMMAND ${MACDEPLOYQT_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/install/bin/schnapps.app -always-overwrite
 #    )

# Packaging stuff
#include(InstallRequiredSystemLibraries)
add_subdirectory(packaging)




#include(packaging/DeployQt.cmake)



# set( CPACK_PACKAGE_CONTACT "Lionel <lionel@plop.org>" )

# set(CPACK_PACKAGE_NAME "SCHNApps")
# set(CPACK_PACKAGE_VENDOR "The SCHNApps Company")
# set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Modeleur geometrique")
# set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
# set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
# set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
# set(CPACK_VERBATIM_VARIABLES YES)

#set(CPACK_PACKAGE_DESCRIPTION_FILE
#set(CPACK_RESOURCE_FILE_WELCOME
#set(CPACK_RESOURCE_FILE_LICENSE
#set(CPACK_RESOURCE_FILE_README


# if(WIN32)
# 	set(CPACK_GENERATOR WIX)
# 	set(CPACK_PACKAGE_INSTALL_DIRECTORY "$PROGRAMFILES64")

# 	set(CPACK_WIX_PROGRAM_MENU_FOLDER "SCHNApps")

# 	set(target "schnapps")

# 	windeployqt(${target})

# 	#set(CPACK_NSIS_ENABLE_UNINSTALL_BEFORE_INSTALL ON)
#     set(CPACK_INSTALLED_DIRECTORIES "${CMAKE_INSTALL_PREFIX};.") #look inside install dir for packaging

#     set(CPACK_PACKAGE_VENDOR "SCHNApps Team")
#     #set(CPACK_NSIS_URL_INFO_ABOUT "https://github.com/CGOGN/SCHNApps")
#     #set(CPACK_NSIS_CONTACT "xxxx@unistra.fr")

#     #set(CPACK_NSIS_PACKAGE_NAME "${target}-${${target}_VERSION}")
#     #set(CPACK_NSIS_DISPLAY_NAME "${target}")
#     set(CPACK_PACKAGE_VERSION "${VERSION}")
#     set(CPACK_BUNDLE_RC_PREFIX "${BUNDLE_RC_PREFIX}")
# endif()


# if(APPLE)
# 	set( CPACK_GENERATOR "DragNDrop" )
#  	set( CPACK_PACKAGE_FILE_NAME "Schnapps-test" )
# 	set(target "SCHNApps")

# 	#macdeployqt(${target})

# endif()

#include(CPack)

