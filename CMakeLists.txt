CMAKE_MINIMUM_REQUIRED(VERSION 2.6.0)

project(qxkb)

set(QXKB_VERSION 0.4.4)
add_definitions(-DQXKB_VERSION="${QXKB_VERSION}")

set(QXKB_TRANSLATION_PATH ${CMAKE_INSTALL_PREFIX}/share/qxkb/i18n)
add_definitions(-DQXKB_TRANSLATION_PATH="${QXKB_TRANSLATION_PATH}")


# find the Qt
set(QT_MIN_VERSION "4.6.0")
set(QT_USE_QTSVG 1)
find_package(Qt4 REQUIRED)
include(${QT_USE_FILE})


# find required X11 modules
find_package(X11 REQUIRED)

if (NOT X11_Xkb_FOUND)
    message(STATUS "Xkb includes: ${X11_Xkblib_INCLUDE_PATH}")
    message(FATAL "X11 Xkb libraries or include not found!")
endif()


include_directories(
	${QT_INCLUDES}
	${X11_Xkblib_INCLUDE_PATH}
	${CMAKE_CURRENT_BINARY_DIR}
	${CMAKE_CURRENT_SOURCE_DIR}/src
)


set (QXKB_SOURCES
	src/qxkb.cpp
	src/xkeyboard.cpp
	src/xkbconf.cpp
	src/x11tools.cpp
	src/main.cpp
	src/datamodels.cpp
)

set (QXKB_MOCS
	src/qxkb.h
	src/xkeyboard.h
	src/xkbconf.h
#	src/x11tools.h
	src/datamodels.h
#	src/defs.h
)

set (QXKB_FORMS
	ui/kswconfig.ui
)

set (QXKB_RESOURCES
	qrc/icon.qrc
)

#set(QXKB_TRANSLATIONS
#	language/qxkb_ru.ts
#)


#QT4_ADD_TRANSLATION(QXKB_QM ${QXKB_TRANSLATIONS})
QT4_WRAP_UI(QXKB_UI ${QXKB_FORMS})
QT4_WRAP_CPP(QXKB_MOC_SRC ${QXKB_MOCS})
QT4_ADD_RESOURCES(QXKB_RCC ${QXKB_RESOURCES})


set(EXE_NAME ${CMAKE_PROJECT_NAME})
add_executable(${EXE_NAME} ${QXKB_SOURCES} ${QXKB_UI} ${QXKB_MOC_SRC} ${QXKB_RCC})
target_link_libraries(${EXE_NAME} ${QT_LIBRARIES} ${X11_X11_LIB} xkbfile)


# installation
#  binary
install(TARGETS ${EXE_NAME} RUNTIME DESTINATION bin LIBRARY DESTINATION lib${LIB_SUFFIX})
#  translations (qm)
install(DIRECTORY language/ DESTINATION ${QXKB_TRANSLATION_PATH}
	FILES_MATCHING PATTERN "*.qm"
	PATTERN ".svn" EXCLUDE)
#  themes
install(DIRECTORY theme/ DESTINATION share/qxkb/theme PATTERN ".svn" EXCLUDE)
#  desktop file
install(FILES desktop/qxkb.desktop DESTINATION share/applications)
#  icon
install(FILES desktop/qxkb.png DESTINATION share/pixmaps)



# "make dist" target
string(TOLOWER ${CMAKE_PROJECT_NAME} CPACK_PACKAGE_NAME)
SET(CPACK_PACKAGE_VERSION "${QXKB_VERSION}")
SET(CPACK_SOURCE_GENERATOR "TBZ2")
SET(CPACK_SOURCE_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}")
SET(CPACK_IGNORE_FILES "/CVS/;/\\\\.svn/;\\\\.swp$;\\\\.#;/#;\\\\.tar.gz$;/CMakeFiles/;CMakeCache.txt;refresh-copyright-and-license.pl;\\\\.git$;build;release;")
SET(CPACK_SOURCE_IGNORE_FILES ${CPACK_IGNORE_FILES})
INCLUDE(CPack)
# simulate autotools' "make dist"
add_custom_target(dist COMMAND ${CMAKE_MAKE_PROGRAM} package_source)
