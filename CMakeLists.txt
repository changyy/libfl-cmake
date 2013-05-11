cmake_minimum_required(VERSION 2.8)
project (libfl)

if (DEFINED CMAKE_TOOLCHAIN_FILE)
	message("Include Cross Platform cmake: " ${CROSS_PLATFORM_CMAKE})
	get_filename_component(COMPILER_NAME ${CMAKE_C_COMPILER} NAME)
	message( "CMAKE_C_COMPILER: ${COMPILER_NAME}")
	string(REGEX REPLACE "(.*)-gcc" "\\1" target_platform "${COMPILER_NAME}")
	message( "target_platform: ${target_platform}")
	set(target_platform_preconfigure_options "CC=${target_platform}-gcc;AR=${target_platform}-ar;LD=${target_platform}-ld;RANLIB=${target_platform}-ranlib;NM=${target_platform}-nm;AS=${target_platform}-as;OBJDUMP=${target_platform}-objdump;OBJCOPY=${target_platform}-objcopy;STRIP=${target_platform}-strip;STRINGS=${target_platform}-strings;SIZE=${target_platform}-size")
endif(DEFINED CMAKE_TOOLCHAIN_FILE)

set(SELF_CPPFLAGS "")
set(SELF_LDFLAGS "")
set(SELF_DEPENDS "")

set(TOOL_LIBFL_SOURCE flex-2.5.37.tar.bz2)
set(TOOL_LIBFL_DIR flex-2.5.37)
set(CONFIG_ADD_OPTIONS "")
	if(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT BUILD_LIBFL
			# source
			#COMMAND wget -qO- http://downloads.sourceforge.net/project/flex/${TOOL_LIBFL_SOURCE} |tar -xjvf - 
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBFL_SOURCE} || wget -qO- http://downloads.sourceforge.net/project/flex/${TOOL_LIBFL_SOURCE} > ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBFL_SOURCE}
			COMMAND tar -xjvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBFL_SOURCE}
			# configure
			COMMAND ${target_platform_preconfigure_options} ${TOOL_LIBFL_DIR}/configure --host="${target_platform}" --prefix=${CMAKE_BINARY_DIR} LDFLAGS="${SELF_LDFLAGS}" CPPFLAGS="${SELF_CPPFLAGS}" ${CONFIG_ADD_OPTIONS} 
			# build
			COMMAND make -j4 && make install
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT "Build libfl"
		)
	else(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT BUILD_LIBFL
			#COMMAND wget -qO- http://downloads.sourceforge.net/project/flex/${TOOL_LIBFL_SOURCE} | tar -xjvf - 
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBFL_SOURCE} || wget -qO- http://downloads.sourceforge.net/project/flex/${TOOL_LIBFL_SOURCE} > ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBFL_SOURCE}
			COMMAND tar -xjvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBFL_SOURCE}
			COMMAND ${TOOL_LIBFL_DIR}/configure --prefix=${CMAKE_BINARY_DIR} LDFLAGS="${SELF_LDFLAGS}" CPPFLAGS="${SELF_CPPFLAGS}" ${CONFIG_ADD_OPTIONS}
			COMMAND make -j4 && make install 
			WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
			COMMENT "Build libfl" 
		)
	endif(DEFINED CMAKE_TOOLCHAIN_FILE)
	add_custom_target(build_libfl ALL DEPENDS ${SELF_DEPENDS} BUILD_LIBFL )
