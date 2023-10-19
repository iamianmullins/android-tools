enable_language(C CXX)

# Use C11 with GNU extensions.
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_EXTENSIONS ON)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Use C++20 with GNU extensions.
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_EXTENSIONS ON)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Android seems to use various attributes supported by clang but not by
# GCC which causes it to emit lots of warnings. Since these attributes
# don't seem to effect runtime behaviour simply disable the warnings.
add_compile_options(-Wno-attributes)

# libfsmgr (required by fastboot) requires a 64-bit off_t for lseek. On
# 32-bit glibc platforms this is not the case by default.
add_compile_definitions(_FILE_OFFSET_BITS=64)

set(android-vendored
	avb
	libbase
	libufdt
	mkbootimg)

# XXX: Consider using https://cmake.org/cmake/help/v3.0/module/ExternalProject.html
if(ANDROID_TOOLS_PATCH_VENDOR AND EXISTS "${ANDROID_PATCH_DIR}/")
	execute_process(COMMAND git submodule --quiet update)
	foreach(v ${android-vendored})
		file(GLOB patches ${ANDROID_PATCH_DIR}/${v}/*.patch)
		if(patches)
			message(STATUS "Applying patches for: ${v}")
			execute_process(COMMAND git -C
				${CMAKE_CURRENT_SOURCE_DIR}/${v} am ${patches}
				RESULT_VARIABLE ret)
			if(NOT "${ret}" STREQUAL "0")
				message(FATAL_ERROR "Couldn't apply patches for ${v}")
			endif()
		endif(patches)
	endforeach(v)
endif()

include(CMakeLists.sparse.txt)
include(CMakeLists.mkbootimg.txt)
include(CMakeLists.libufdt.txt)
include(CMakeLists.avb.txt)

# Targets which should be installed by `make install`.
install(TARGETS
	append2simg
	img2simg
	simg2img
	DESTINATION bin)

# Install license files.
# Disabled for now, see https://github.com/nmeum/android-tools/issues/30#issuecomment-855365636
#set(LICENSE_DIR "${CMAKE_INSTALL_FULL_DATADIR}/licenses/android-tools")
#install(FILES core/NOTICE RENAME AOSP_LICENSE DESTINATION "${LICENSE_DIR}")
