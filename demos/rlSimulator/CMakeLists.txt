if(MSVC AND CMAKE_CXX_COMPILER_VERSION VERSION_LESS "19.00.23918")
	find_package(Boost REQUIRED)
endif()

find_package(OpenGL REQUIRED)

if(USE_QT5)
	find_package(Qt5 COMPONENTS Core Gui Network OpenGL Widgets QUIET)
endif()

if(Qt5_FOUND)
	set(QT_LIBRARIES ${Qt5Core_LIBRARIES} ${Qt5Gui_LIBRARIES} ${Qt5OpenGL_LIBRARIES} ${Qt5Network_LIBRARIES} ${Qt5Widgets_LIBRARIES})
	set(QT_FOUND Qt5_FOUND)
else()
	set(QT_USE_IMPORTED_TARGETS ON)
	find_package(Qt4 COMPONENTS QtCore QtGui QtNetwork QtOpenGL)
	set(QT_USE_QTMAIN ON)
	include(${QT_USE_FILE})
endif()

set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

find_package(SoQt)

if(QT_FOUND AND SoQt_FOUND)
	set(
		HDRS
		ConfigurationDelegate.h
		ConfigurationModel.h
		MainWindow.h
		OperationalDelegate.h
		OperationalModel.h
		Server.h
		Socket.h
	)
	
	set(
		SRCS
		rlSimulator.cpp
		ConfigurationDelegate.cpp
		ConfigurationModel.cpp
		MainWindow.cpp
		OperationalDelegate.cpp
		OperationalModel.cpp
		Server.cpp
		Socket.cpp
		${rl_SOURCE_DIR}/robotics-library.rc
	)
	
	add_executable(
		rlSimulator
		WIN32
		${HDRS}
		${SRCS}
	)
	
	target_compile_definitions(
		rlSimulator
		PUBLIC
		${SoQt_DEFINITIONS}
	)
	
	if(MSVC AND CMAKE_CXX_COMPILER_VERSION VERSION_LESS "19.00.23918")
		target_compile_definitions(
			rlSimulator
			PUBLIC
			-DBOOST_ALL_NO_LIB
			-DBOOST_CHRONO_HEADER_ONLY
			-DBOOST_ERROR_CODE_HEADER_ONLY
			-DBOOST_SYSTEM_NO_DEPRECATED
		)
	endif()
	
	target_include_directories(
		rlSimulator
		PUBLIC
		${Boost_INCLUDE_DIR}
		${Coin_INCLUDE_DIRS}
		${SoQt_INCLUDE_DIRS}
	)
	
	target_link_libraries(
		rlSimulator
		math
		mdl
		sg
		${Coin_LIBRARIES}
		${QT_LIBRARIES}
		${SoQt_LIBRARIES}
	)
	
	set_target_properties(
		rlSimulator
		PROPERTIES
		VERSION ${VERSION}
	)
	
	if(WIN32)
		set_target_properties(
			rlSimulator
			PROPERTIES
			DEBUG_POSTFIX d
		)
	endif()
	
	install(
		TARGETS rlSimulator
		COMPONENT demos
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	)
	
	if(MSVC AND BUILD_SHARED_LIBS AND NOT CMAKE_VERSION VERSION_LESS 3.1)
		install(FILES $<TARGET_PDB_FILE:rlSimulator> DESTINATION ${CMAKE_INSTALL_BINDIR} CONFIGURATIONS Debug RelWithDebInfo COMPONENT demos)
	endif()
	
	if(UNIX)
		configure_file(rlSimulator.desktop.in rlSimulator.desktop @ONLY)
		install(FILES ${CMAKE_CURRENT_BINARY_DIR}/rlSimulator.desktop DESTINATION ${CMAKE_INSTALL_DATADIR}/applications COMPONENT demos)
	endif()
	
	set(
		CPACK_NSIS_CREATE_ICONS_EXTRA
		${CPACK_NSIS_CREATE_ICONS_EXTRA}
		"CreateShortCut \\\\
			\\\"$SMPROGRAMS\\\\$STARTMENU_FOLDER\\\\rlSimulator.lnk\\\" \\\\
			\\\"$INSTDIR\\\\${CMAKE_INSTALL_BINDIR}\\\\rlSimulator.exe\\\" \\\\
			\\\"\\\\
				\\\$\\\\\\\"$INSTDIR\\\\${CMAKE_INSTALL_DATADIR}\\\\rl-${VERSION}\\\\examples\\\\rlsg\\\\unimation-puma560_boxes.xml\\\$\\\\\\\" \\\\
				\\\$\\\\\\\"$INSTDIR\\\\${CMAKE_INSTALL_DATADIR}\\\\rl-${VERSION}\\\\examples\\\\rlmdl\\\\unimation-puma560.xml\\\$\\\\\\\"\\\\
			\\\" \\\\
			\\\"\\\" \\\"\\\" \\\"\\\" \\\"\\\" \\\\
			\\\"Simulation demo with Unimation Puma 560\\\""
		PARENT_SCOPE
	)
	set(CPACK_NSIS_DELETE_ICONS_EXTRA ${CPACK_NSIS_DELETE_ICONS_EXTRA} "Delete \\\"$SMPROGRAMS\\\\$START_MENU\\\\rlSimulator.lnk\\\"" PARENT_SCOPE)
endif()

