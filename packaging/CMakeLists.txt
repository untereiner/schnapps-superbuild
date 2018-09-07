
set(VERSION 0.1)


 set(CPACK_GENERATOR "WIX") # this need WiX Tooset installed and a path to candle.exe

# Packaging and install. Warning : not for faint of the heart
set(CPACK_PACKAGE_NAME "Lionel Untereiner")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "A geometric modeling tool based on combinatorial maps")
set(CPACK_PACKAGE_VENDOR "geosiris.com")
set(CPACK_PACKAGE_CONTACT http://www.geosiris.com)
##set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_SOURCE_DIR}/README.md")
set(CPACK_PACKAGE_VERSION ${VERSION})
set(CPACK_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}-installer")

# Source package should always be named the same regardless of the platform on which we build
set(CPACK_SOURCE_PACKAGE_FILE_NAME "schnapps-${VERSION}")
set(CPACK_SOURCE_STRIP_FILES true)
set(CPACK_STRIP_FILES true)
##set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/COPYING.txt")
##set(CPACK_RESOURCE_FILE_README "${CMAKE_SOURCE_DIR}/README.md")


set(CPACK_SOURCE_IGNORE_FILES "${CMAKE_SOURCE_DIR}/build/;${CMAKE_SOURCE_DIR}/.git/")

if(APPLE)
	find_package(Qt5Core REQUIRED)
	get_target_property(_qmake_executable Qt5::qmake IMPORTED_LOCATION)
	get_filename_component(_qt_bin_dir "${_qmake_executable}" DIRECTORY)
	find_program(MACDEPLOYQT_EXECUTABLE macdeployqt HINTS "${_qt_bin_dir}")
	if(APPLE AND NOT MACDEPLOYQT_EXECUTABLE)
	    message(FATAL_ERROR "macdeployqt not found")
	else()
		message(STATUS "${MACDEPLOYQT_EXECUTABLE}")
	endif()

    
    #ExternalProject_Add_Step(schnapps post_build 
    #    COMMENT "Deploying Qt..."
    #	DEPENDEES build 
    #	DEPENDERS install
    #    COMMAND "${MACDEPLOYQT_EXECUTABLE}"
    #        \"$<TARGET_FILE_DIR:schnapps>/../..\"
    #        -always-overwrite
    #)

	#macdeployqt(SCHNApps)

endif()

if(WIN32)
	set(CPACK_MONOLITHIC_INSTALL 1)
	set(CPACK_NSIS_ENABLE_UNINSTALL_BEFORE_INSTALL 1)
	set(CPACK_NSIS_MUI_FINISHPAGE_RUN "tagainijisho")
	set(CPACK_PACKAGE_INSTALL_DIRECTORY "Tagaini Jisho")
	set(CPACK_NSIS_EXECUTABLES_DIRECTORY ".")
	set(CPACK_PACKAGE_EXECUTABLES "tagainijisho" "Tagaini Jisho")
	set(CPACK_NSIS_INSTALLED_ICON_NAME "tagainijisho")
	#set(CPACK_PACKAGE_ICON ${CMAKE_SOURCE_DIR}/images/appicon.ico)
	set(CPACK_NSIS_INSTALLED_ICON_NAME ${CMAKE_SOURCE_DIR}/images/appicon.ico)

	set(DB_DIR ".")
	set(I18N_DIR "./i18n")
	set(DOC_DIR "./doc")
	set(SYS_QT_I18N_DIR "${CMAKE_INSTALL_PREFIX}/share/qt/translations")


endif()

include(CPack)

unset(VERSION)