# Copyright (c) Martin Schweiger
# Licensed under the MIT License

set(MESH_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/Meshes)
set(TEXTURE_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/Textures)
set(CONFIG_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/Config)

add_library(Atlantis SHARED
	AscentAP.cpp
	Atlantis.cpp
	PlBayOp.cpp
	Atlantis.rc
	../Common.cpp
	${ORBITER_SOURCE_MODULE_DIR}/Common/Dialog/Graph.cpp
	${ORBITER_SOURCE_MODULE_DIR}/Common/Dialog/TabDlg.cpp
)

set_target_properties(Atlantis
	PROPERTIES
	FOLDER Vessels/Atlantis
)

target_include_directories(Atlantis
	PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}
	PUBLIC ${ORBITER_SOURCE_SDK_INCLUDE_DIR}
	PUBLIC ${ORBITER_SOURCE_MODULE_DIR}
	PUBLIC ${CMAKE_CURRENT_BINARY_DIR}                 # for the dynamically generated header files
)

target_link_libraries(Atlantis
	${ORBITER_LIB}
	${ORBITER_SDK_LIB}
	${ORBITER_DLGCTRL_LIB}
	UxTheme.lib
	$<TARGET_LINKER_FILE:Atlantis_Tank>
)

# scan mesh files for labels and output to header file
add_custom_command(OUTPUT meshres.h
	COMMAND ${ORBITER_TOOL_MESHC} /I ${MESH_SOURCE_DIR}/Atlantis.msh /P - /O meshres.h
	DEPENDS ${MESH_SOURCE_DIR}/Atlantis.msh
)
add_custom_command(OUTPUT meshres_vc.h
	COMMAND ${ORBITER_TOOL_MESHC} /I ${MESH_SOURCE_DIR}/AtlantisVC.msh /P _VC /O meshres_vc.h
	DEPENDS ${MESH_SOURCE_DIR}/AtlantisVC.msh
)
add_custom_target(Atlantis_Prerequisites
	DEPENDS meshres.h meshres_vc.h
)
add_dependencies(Atlantis_Prerequisites
	meshc
)

add_custom_command(
	TARGET Atlantis PRE_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy_directory ${CONFIG_SOURCE_DIR}/ ${CMAKE_BINARY_DIR}/Config/Vessels
)

add_custom_command(
	TARGET Atlantis PRE_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy_directory ${MESH_SOURCE_DIR}/ ${CMAKE_BINARY_DIR}/Meshes/Atlantis
)

add_custom_command(
	TARGET Atlantis PRE_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy_directory ${TEXTURE_SOURCE_DIR}/ ${ORBITER_BINARY_TEXTURES_DIR}/Atlantis
)

add_dependencies(Atlantis
	${OrbiterTgt}
	Orbitersdk
	DlgCtrl
	Atlantis_SRB
	Atlantis_Tank
	Atlantis_Prerequisites
)

set_target_properties(Atlantis_Prerequisites
	PROPERTIES
	FOLDER Vessels/Atlantis
)

# Installation
install(TARGETS Atlantis
	RUNTIME
	DESTINATION ${ORBITER_INSTALL_MODULE_DIR}
)
install(DIRECTORY ${CONFIG_SOURCE_DIR}/
	DESTINATION ${ORBITER_INSTALL_ROOT_DIR}/Config/Vessels
)
install(DIRECTORY ${MESH_SOURCE_DIR}/
	DESTINATION ${ORBITER_INSTALL_ROOT_DIR}/Meshes/Atlantis
)
install(DIRECTORY ${TEXTURE_SOURCE_DIR}/
	DESTINATION ${ORBITER_INSTALL_TEXTURES_DIR}/Atlantis
)
