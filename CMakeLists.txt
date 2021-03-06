CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

# Define project
PROJECT("mmlinuxserver")

# Project version
SET(CMAKE_BUILD_TYPE Release)

# Build parameters
SET(TARGET_NAME "mmserver")
FILE(GLOB_RECURSE SOURCE_FILES src/*.cpp)
SET(CMAKE_CXX_FLAGS_RELEASE "-s -Wall -Wextra -Wconversion")

SET(MMSERVER_VERSION_MAJOR 1)
SET(MMSERVER_VERSION_MINOR 4)
SET(MMSERVER_VERSION_PATCH 0)

CONFIGURE_FILE(
	"${CMAKE_SOURCE_DIR}/src/version.hpp"
	"${CMAKE_CURRENT_BINARY_DIR}/version.hpp.in"
)

FIND_PACKAGE(PkgConfig)
PKG_CHECK_MODULES(GTK gtk+-2.0)

# Build of the program
ADD_EXECUTABLE(${TARGET_NAME} ${SOURCE_FILES})
TARGET_LINK_LIBRARIES(${TARGET_NAME}
	X11
	Xtst
	Xmu
	pcrecpp
	config++
	pthread
	avahi-common
	avahi-client
	${GTK_LIBRARIES}
)
INCLUDE_DIRECTORIES(
	${CMAKE_CURRENT_BINARY_DIR}
	${GTK_INCLUDE_DIRS}
)

SET(CPACK_GENERATOR "DEB")
SET(CPACK_SET_DESTDIR "ON")
SET(CPACK_PACKAGE_VERSION "${MMSERVER_VERSION_MAJOR}.${MMSERVER_VERSION_MINOR}.${MMSERVER_VERSION_PATCH}")
SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "Jim DeVona")
SET(CPACK_DEBIAN_PACKAGE_HOMEPAGE "https://github.com/anoved/mmserver/")
SET(CPACK_DEBIAN_PACKAGE_SECTION "X11")
SET(CPACK_DEBIAN_PACKAGE_DESCRIPTION "Modified Mobile Mouse Server for Linux")
SET(CPACK_DEBIAN_PACKAGE_DEPENDS "libpcrecpp0, libconfig++8")
SET(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA "${CMAKE_SOURCE_DIR}/share/postinst;${CMAKE_SOURCE_DIR}/share/prerm")

# fix permissions of postinst/prerm
EXEC_PROGRAM(chmod ARGS 0755 "${CMAKE_SOURCE_DIR}/share/postinst" "${CMAKE_SOURCE_DIR}/share/prerm")

# detect ubuntu version...
EXEC_PROGRAM(lsb_release ARGS -c -s OUTPUT_VARIABLE CPACK_SYSTEM_NAME RETURN_VALUE CPACK_SYSTEM_NAME_RET)
IF(CPACK_SYSTEM_NAME_RET GREATER 0)
	SET(CPACK_SYSTEM_NAME "linux")
ENDIF(CPACK_SYSTEM_NAME_RET GREATER 0)
SET(CPACK_SYSTEM_NAME "${CPACK_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")

INCLUDE(CPack)

# Installation of the program
INSTALL(TARGETS ${TARGET_NAME}
	DESTINATION "/usr/sbin"
	PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)

# Installation of the configuration file
INSTALL(PROGRAMS "${CMAKE_SOURCE_DIR}/mmserver.conf"
	DESTINATION "/usr/share/mmserver"
	PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ)

# Installation of the auto-run configuration
INSTALL(PROGRAMS "${CMAKE_SOURCE_DIR}/share/mmserver.desktop"
	DESTINATION "/usr/share/gnome/autostart"
	PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ)

# Installation of Gnome menu item
INSTALL(PROGRAMS "${CMAKE_SOURCE_DIR}/share/mmserver.desktop"
	DESTINATION "/usr/share/applications"
	PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ)

# Installation of icon (used for menu item)
INSTALL(PROGRAMS "${CMAKE_SOURCE_DIR}/icons/mm-idle.svg"
	DESTINATION "/usr/share/mmserver/icons"
	PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ)
