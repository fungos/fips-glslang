#
# project: fips-glslang
#

if (NOT FIPS_IMPORT)
    cmake_minimum_required(VERSION 2.8)
    get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
    include("${FIPS_ROOT_DIR}/cmake/fips.cmake")
    fips_setup()
    fips_project(fips-glslang)
endif()

if (CMAKE_TOOLCHAIN_FILE)
	message(ERROR Cross-compiling not supported)
endif() # CMAKE_TOOLCHAIN_FILE

#-------------------------------------------------------------------------------

option(ENABLE_AMD_EXTENSIONS "Enables support of AMD-specific extensions" ON)
option(ENABLE_NV_EXTENSIONS "Enables support of Nvidia-specific extensions" ON)

include_directories(glslang/)
if(ENABLE_AMD_EXTENSIONS)
    add_definitions(-DAMD_EXTENSIONS)
endif(ENABLE_AMD_EXTENSIONS)

if(ENABLE_NV_EXTENSIONS)
    add_definitions(-DNV_EXTENSIONS)
endif(ENABLE_NV_EXTENSIONS)

if(CMAKE_COMPILER_IS_GNUCXX)
	add_definitions(-Wall -Wmaybe-uninitialized -Wuninitialized -Wunused -Wunused-local-typedefs
		-Wunused-parameter -Wunused-value  -Wunused-variable -Wunused-but-set-parameter -Wunused-but-set-variable)
	add_definitions(-Wno-reorder)  # disable this from -Wall, since it happens all over.
	add_definitions(-std=c++11)
elseif(${CMAKE_CXX_COMPILER_ID} MATCHES "Clang")
	add_definitions(-Wall -Wuninitialized -Wunused -Wunused-local-typedefs
		-Wunused-parameter -Wunused-value  -Wunused-variable)
	add_definitions(-Wno-reorder)  # disable this from -Wall, since it happens all over.
	add_definitions(-std=c++11)
endif()

#-[ libs ]---------------------------------------------------------------------

fips_begin_lib(OGLCompiler)
    fips_include_directories(glslang/OGLCompilersDLL/)
    fips_src(glslang/OGLCompilersDLL GROUP "OGLCompilersDLL")
fips_end_lib()

fips_begin_lib(SPIRV)
    fips_include_directories(glslang/SPIRV/)
    fips_src(glslang/SPIRV GROUP "SPIRV")
fips_end_lib()

fips_begin_lib(hlsl)
    fips_include_directories(glslang/hlsl/)
    fips_src(glslang/hlsl 	 GROUP "hlsl")
fips_end_lib()

fips_begin_lib(glslang)
    fips_include_directories(glslang/glslang/)
    fips_src(glslang/glslang EXCEPT OSDependent/* GROUP "glslang")
fips_end_lib()

fips_begin_lib(OSDependent)
    fips_include_directories(glslang/glslang/)
    if (FIPS_MACOS OR FIPS_LINUX)
    	fips_include_directories(glslang/glslang/OSDependent/Unix/)
        fips_src(glslang/glslang/OSDependent/Unix/)
    elseif (FIPS_WINDOWS)
   		fips_include_directories(glslang/glslang/OSDependent/Windows/)
        fips_src(glslang/glslang/OSDependent/Windows/)
    else()
        message(ERROR "Unsupported platform")
    endif()
fips_end_lib()

#-[ execs ]---------------------------------------------------------------------

fips_ide_group(Tools)
fips_begin_app(glslangValidator cmdline)
    fips_include_directories(glslang/StandAlone/)
    fips_files(glslang/StandAlone/ResourceLimits.cpp)
    fips_files(glslang/StandAlone/StandAlone.cpp)
    fips_deps(glslang hlsl OGLCompiler OSDependent SPIRV)

    if (FIPS_WINDOWS)
        add_definitions(-D__STDC__ -D__STDC_VERSION__=199901L -Dstrdup=_strdup -Dalloca=_alloca -Disascii=__isascii)
        fips_libs(psapi)
    elseif (FIPS_LINUX OR FILS_MACOS)
    	fips_libs(pthread)
    endif()
fips_end_app()
	
fips_begin_app(spirv-remap cmdline)
    fips_include_directories(glslang/StandAlone/)
    fips_files(glslang/StandAlone/ResourceLimits.cpp)
    fips_files(glslang/StandAlone/spirv-remap.cpp)
    fips_deps(glslang hlsl OGLCompiler OSDependent SPIRV)

    if (FIPS_WINDOWS)
        add_definitions(-D__STDC__ -D__STDC_VERSION__=199901L -Dstrdup=_strdup -Dalloca=_alloca -Disascii=__isascii)
        fips_libs(psapi)
    elseif (FIPS_LINUX OR FILS_MACOS)
    	fips_libs(pthread)
    endif()
fips_end_app()

#-------------------------------------------------------------------------------
