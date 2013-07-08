CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
#################### Project Settings ####################
SET(PROJECT_NAME "robomongo" CACHE STRING "Project Name")
SET(${PROJECT_NAME}_PROJECT_DOMAIN "www.robomongo.com" CACHE STRING "Internet domain name for ${ProjectName}")
SET(PROJECT_COMPANYNAME "paralect")
##########################################################
STRING(TOLOWER ${PROJECT_NAME} PROJECT_NAME_LOWERCASE)
SET (CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake")
########################## Options #########################
OPTION(GENERATE_TRANSLATIONS "Generate ts-files for ${PROJECT_NAME}" OFF)
OPTION(MANIFEST_ENABLE "Enable Manifest" OFF)
#############################################################
# Configure project
PROJECT(${PROJECT_NAME_LOWERCASE})
set(MAJOR "0")
set(MINOR "7")
set(PATCH "1")
set(BUILD "0")
set(${PROJECT_NAME}_VERSION "${MAJOR}.${MINOR}.${PATCH}.${BUILD} beta" CACHE STRING "${PROJECT_NAME} version")
OPTION(BOOST_ENABLED "Enable boost system support" OFF)
OPTION(BUILD_PCRECPP_LIBRARY "Build pcrecpp library from sources" ON)
OPTION(QT_ENABLED "Enable Qt support" ON)
OPTION(UNICODE_ENABLE "Support unicode" OFF)
OPTION(DEVELOPER_DYNAMIC_RUNTIME "Build with dynamic C++ runtime" ON)
OPTION(FORCE_BUILD "Build in force mode used only for comand line." OFF)
IF(MSVC)
	IF(CMAKE_CL_64)
		OPTION(BUILD_64X "Build 64bit project" ON)
	ELSE()
		OPTION(BUILD_64X "Build 64bit project" OFF)
	ENDIF()
ELSE()
	OPTION(BUILD_64X "Build 64bit project" OFF)
ENDIF()
INCLUDE(cmake/config.cmake)

DEFINE_DEFAULT_DEFINITIONS()
SETUP_COMPILER_SETTINGS(${DEVELOPER_DYNAMIC_RUNTIME})

# Search libraries
# use Qt
DETECT_QT()
##################### Libraries search ###################
if(DEVELOPER_QT5)
SET(QT_COMPONENTS_TO_USE 
	Qt5Core 
	Qt5Gui 
	Qt5Widgets
)
ELSE()
SET(QT_COMPONENTS_TO_USE 
	QtCore 
	QtGui 
)
ENDIF()
INTEGRATE_QT(${QT_COMPONENTS_TO_USE})
##########################################################

## Qt static or dinamic detector ##

# enable GIF and ICO support
IF(WINDOWS)
	FIND_LIBRARY(QT_ICO_PLUGIN qico PATHS ${QT_PLUGINS_DIR} PATH_SUFFIXES imageformats)
    FIND_LIBRARY(QT_ICO_PLUGIN_DEBUG qicod PATHS ${QT_PLUGINS_DIR} PATH_SUFFIXES imageformats)
ELSEIF(MAC_OS)
    FIND_LIBRARY(QT_ICO_PLUGIN qico PATHS ${QT_PLUGINS_DIR} PATH_SUFFIXES imageformats)
    FIND_LIBRARY(QT_ICO_PLUGIN_DEBUG qico_debug PATHS ${QT_PLUGINS_DIR} PATH_SUFFIXES imageformats)
ENDIF()

#IF(MINGW OR CMAKE_COMPILER_IS_GNUCXX)
#    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -fpermissive")
#IF(CMAKE_COMPILER_IS_GNUCC)
#	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -lpthread -fPIC")
#ENDIF()
#ENDIF(MINGW OR CMAKE_COMPILER_IS_GNUCXX)
IF(CMAKE_COMPILER_IS_GNUCXX)
	IF(NOT MINGW)
		SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
	ENDIF()
ENDIF()

IF(BUILD_64X)
	IF(CMAKE_COMPILER_IS_GNUCXX)
		SET(CFLAGS "${CFLAGS} -m64")
		SET(CXXFLAGS "${CXXFLAGS} -m64")
	ELSEIF(MSVC)
		SET(CMAKE_EXE_LINKER_FLAGS "/machine:x64")
	ENDIF()
	IF(APPLE)
                SET(CMAKE_OSX_ARCHITECTURES x86_64)
	ENDIF()
	SET(ARCH_TEXT 64)
ELSE()
	IF(CMAKE_COMPILER_IS_GNUCXX)
		SET(CFLAGS "${CFLAGS} -m32")
		SET(CXXFLAGS "${CXXFLAGS} -m32")
	ELSEIF(MSVC)
		set(CMAKE_EXE_LINKER_FLAGS "/machine:x86")
	ENDIF()
	IF(APPLE)
                SET(CMAKE_OSX_ARCHITECTURES i386)
	#-DCMAKE_OSX_ARCHITECTURES=x86_64;i386
	ENDIF()
	SET(ARCH_TEXT 32)
ENDIF()

if(WIN32 OR APPLE)
	SET(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/bin")
endif(WIN32 OR APPLE)
############################################################################

IF(MANIFEST_ENABLE)
	IF(WIN32)
                ADD_CUSTOM_COMMAND(TARGET ${PROJECT_NAME_LOWERCASE} POST_BUILD COMMAND mt -manifest "${CMAKE_CURRENT_SOURCE_DIR}/resources/win7.manifest" -outputresource:"$(TargetDir)$(TargetFileName)"\;\#1)
	ENDIF(WIN32)
ENDIF(MANIFEST_ENABLE)

IF("${PROJECT_SOURCE_DIR}" STREQUAL "${PROJECT_BINARY_DIR}")
MESSAGE(SEND_ERROR "In-source builds are not allowed.")
ENDIF("${PROJECT_SOURCE_DIR}" STREQUAL "${PROJECT_BINARY_DIR}")
# project sources
ADD_SUBDIRECTORY(src)