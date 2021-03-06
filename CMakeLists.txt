#-------------------------------------------------------------------------------
# Simbody
#
# This is the master CMake file that coordinates
# the build of Simbody. There are four steps:
#    (1) Get files needed for particular platform
#    (2) Build SimTKcommon library
#    (3) Build SimTKmath library
#    (4) Build SimTKsimbody library
#    (5) Build examples
#
#-------------------------------------------------------------------------------
cmake_minimum_required(VERSION 2.8.8)

if(COMMAND cmake_policy)
    cmake_policy(SET CMP0003 NEW)
    cmake_policy(SET CMP0005 NEW)
    if(NOT (${CMAKE_VERSION} VERSION_LESS 3.0))
        # MACOSX_RPATH enabled by default; policy introduced with cmake 3.0.
        cmake_policy(SET CMP0042 NEW)
    endif()
endif(COMMAND cmake_policy)

PROJECT(Simbody)

# At this point CMake will have set CMAKE_INSTALL_PREFIX to /usr/local on Linux
# or appropriate ProgramFiles directory on Windows, for example
# C:/Program Files/Simbody, C:/Program Files (x86)/Simbody, or the local
# language equivalent.

IF(WIN32)
  set (CMAKE_INSTALL_DOCDIR doc CACHE PATH "documentation root (DATAROOTDIR/doc/PROJECT_NAME)")
ELSE()
  # Redefine DOCDIR to use the project name in lowercase to avoid
  # problems with some platforms: NTFS on Win, XFS or JFS variants
  set (CMAKE_INSTALL_DOCDIR share/doc/simbody CACHE PATH "documentation root (DATAROOTDIR/doc/PROJECT_NAME)")
ENDIF()

# Include GNUInstallDirs to get canonical paths
include(GNUInstallDirs)

SET(SIMBODY_MAJOR_VERSION 3)
SET(SIMBODY_MINOR_VERSION 6)
SET(SIMBODY_PATCH_VERSION 0)

SET(SIMBODY_COPYRIGHT_YEARS "2005-15")

# underbar separated list of dotted authors, no spaces or commas
SET(SIMBODY_AUTHORS "Michael.Sherman_Peter.Eastman")

# Report the version number to the CMake UI. Don't include the
# build version if it is zero.
SET(PATCH_VERSION_STRING)
IF(SIMBODY_PATCH_VERSION)
    SET(PATCH_VERSION_STRING ".${SIMBODY_PATCH_VERSION}")
ENDIF()

SET(SIMBODY_VERSION
    "${SIMBODY_MAJOR_VERSION}.${SIMBODY_MINOR_VERSION}${PATCH_VERSION_STRING}"
    CACHE STRING
    "This is the version that will be built (can't be changed in GUI)."
    FORCE)

SET(SIMBODY_SONAME_VERSION
    "${SIMBODY_MAJOR_VERSION}.${SIMBODY_MINOR_VERSION}"
    CACHE STRING
    "Soname version; appended to names of shared libs
    (can't be changed in GUI)."
    FORCE)

# This is the suffix if we're building and depending on versioned libraries.
SET(VN "_${SIMBODY_VERSION}")

SET(BUILD_BINARY_DIR ${CMAKE_BINARY_DIR}
    CACHE PATH
    "The Simbody build (not the install) puts all the libraries and executables together here (with /Release, etc. appended on some platforms).")

# Make everything go in the same binary directory. (These are CMake-defined
# variables.)
SET(EXECUTABLE_OUTPUT_PATH ${BUILD_BINARY_DIR})
SET(LIBRARY_OUTPUT_PATH ${BUILD_BINARY_DIR})

# Static libraries, tests, and examples won't be built unless this
# is set.
SET(BUILD_STATIC_LIBRARIES FALSE CACHE BOOL
"Build '_static' versions of libraries in addition to dynamic libraries?")

# Use this to generate a private set of libraries whose names
# won't conflict with installed versions.
SET(BUILD_USING_NAMESPACE "" CACHE STRING
    "All library names will be prefixed with 'xxx_' if this is set to xxx.")

SET(BUILD_UNVERSIONED_LIBRARIES TRUE CACHE BOOL
 "Build library names, and assume dependency names, with no version numbers?")

SET(BUILD_VERSIONED_LIBRARIES FALSE CACHE BOOL
 "Build library names, and assume dependency names, with version numbers?")

SET(NS)
IF(BUILD_USING_NAMESPACE)
    SET(NS "${BUILD_USING_NAMESPACE}_")
ENDIF(BUILD_USING_NAMESPACE)

# Ensure that debug libraries have "_d" appended to their names.
SET(CMAKE_DEBUG_POSTFIX "_d")

#
# These are the names of all the libraries we may generate. These are
# target names so can be used to specify dependencies of one library
# on another. (In Debug mode the actual targets will have "_d" appended.)
#

SET(SimTKSIMBODY_LIBRARY_NAME ${NS}SimTKsimbody CACHE STRING
"Base name of the library being built; can't be changed here; see BUILD_USING_NAMESPACE variable."
FORCE)
SET(SimTKMATH_LIBRARY_NAME ${NS}SimTKmath CACHE STRING
"Base name of the library being built; can't be changed here; see BUILD_USING_NAMESPACE variable."
FORCE)
SET(SimTKCOMMON_LIBRARY_NAME ${NS}SimTKcommon CACHE STRING
"Base name of the library being built; can't be changed here; see BUILD_USING_NAMESPACE variable."
FORCE)


SET(SimTKCOMMON_SHARED_LIBRARY ${SimTKCOMMON_LIBRARY_NAME})
SET(SimTKCOMMON_STATIC_LIBRARY ${SimTKCOMMON_LIBRARY_NAME}_static)

SET(SimTKCOMMON_LIBRARY_NAME_VN ${NS}SimTKcommon${VN})
SET(SimTKCOMMON_SHARED_LIBRARY_VN ${SimTKCOMMON_LIBRARY_NAME_VN})
SET(SimTKCOMMON_STATIC_LIBRARY_VN ${SimTKCOMMON_LIBRARY_NAME_VN}_static)

SET(SimTKMATH_SHARED_LIBRARY ${SimTKMATH_LIBRARY_NAME})
SET(SimTKMATH_STATIC_LIBRARY ${SimTKMATH_LIBRARY_NAME}_static)

SET(SimTKMATH_LIBRARY_NAME_VN ${NS}SimTKmath${VN})
SET(SimTKMATH_SHARED_LIBRARY_VN ${SimTKMATH_LIBRARY_NAME_VN})
SET(SimTKMATH_STATIC_LIBRARY_VN ${SimTKMATH_LIBRARY_NAME_VN}_static)

SET(SimTKSIMBODY_SHARED_LIBRARY ${SimTKSIMBODY_LIBRARY_NAME})
SET(SimTKSIMBODY_STATIC_LIBRARY ${SimTKSIMBODY_LIBRARY_NAME}_static)

SET(SimTKSIMBODY_LIBRARY_NAME_VN ${NS}SimTKsimbody${VN})
SET(SimTKSIMBODY_SHARED_LIBRARY_VN ${SimTKSIMBODY_LIBRARY_NAME_VN})
SET(SimTKSIMBODY_STATIC_LIBRARY_VN ${SimTKSIMBODY_LIBRARY_NAME_VN}_static)


# Caution: this variable is automatically created by the CMake
# ENABLE_TESTING() command, but we'll take it over here for
# our own purposes too.
SET(BUILD_TESTING ON CACHE BOOL
    "Control building of Simbody test programs.
    To actually build tests, one
    or both of BUILD_TESTS_AND_EXAMPLES_STATIC and
    BUILD_TESTS_AND_EXAMPLES_SHARED must be ON.")

SET(BUILD_EXAMPLES ON CACHE BOOL
    "Control building of Simbody example programs.
    To actually build examples, one
    or both of BUILD_TESTS_AND_EXAMPLES_STATIC and
    BUILD_TESTS_AND_EXAMPLES_SHARED must be ON.")

# Set whether to build the Visualizer code.
SET(BUILD_VISUALIZER ON CACHE BOOL
    "Control building of the visualizer component")

# Turning this off reduces the build time (and space) substantially,
# but you may miss the occasional odd bug. Also currently on Windows it
# is easier to debug the static tests than the DLL-linked ones.
SET(BUILD_TESTS_AND_EXAMPLES_STATIC ON CACHE BOOL
    "If BUILDING_STATIC_LIBRARIES and BUILD_TESTING or BUILD_EXAMPLES, build
    statically-linked test and example programs too? On Windows,
    statically-linked tests may be easier to debug than DLL-linked tests.
    Statically-linked examples are never installed.")
MARK_AS_ADVANCED(BUILD_TESTS_AND_EXAMPLES_STATIC)

SET(BUILD_TESTS_AND_EXAMPLES_SHARED ON CACHE BOOL
    "If BUILD_TESTING or BUILD_EXAMPLES, build dynamically-linked ones?")
MARK_AS_ADVANCED(BUILD_TESTS_AND_EXAMPLES_SHARED)

IF(BUILD_TESTING AND NOT (BUILD_TESTS_AND_EXAMPLES_STATIC OR
        BUILD_TESTS_AND_EXAMPLES_SHARED))
    MESSAGE(SEND_ERROR "No tests would be built, despite BUILD_EXAMPLES"
        "being on, because BUILD_TESTS_AND_EXAMPLES_STATIC and "
        "BUILD_TESTS_AND_EXAMPLES_SHARED are both off.")
ENDIF()

IF(BUILD_EXAMPLES AND NOT (BUILD_TESTS_AND_EXAMPLES_STATIC OR
        BUILD_TESTS_AND_EXAMPLES_SHARED))
    MESSAGE(SEND_ERROR
        "No examples would be built, despite BUILD_EXAMPLES being on, "
        "because BUILD_TESTS_AND_EXAMPLES_STATIC and "
        "BUILD_TESTS_AND_EXAMPLES_SHARED are both off.")
ENDIF()

# In addition to the platform name we need to know the Application Binary
# Interface (ABI) we're building for. Currently that is either x86, meaning
# 32 bit Intel instruction set, or x64 for 64 bit Intel instruction set.

IF(${CMAKE_SIZEOF_VOID_P} EQUAL 8)
    SET(PLATFORM_ABI x64)
ELSE()
    SET(PLATFORM_ABI x86)
ENDIF()

SET(BUILD_PLATFORM "${CMAKE_HOST_SYSTEM_NAME}:${PLATFORM_ABI}" CACHE STRING
    "This is the platform and ABI we're building for. Not changeable here; use a different CMake generator instead."
    FORCE)

# If CMAKE_INSTALL_PREFIX is /usr/local, then the LIBDIR should necessarily be
# lib/. Sometimes (on Linux), LIBDIR is something like x86_64-linux-gnu. The
# linker will search /usr/lib/x86_64-linux-gnu (this path is in
# /etc/ld.so.conf.d), but it will NOT search /usr/local/lib/x86-64-linux-gnu.
# HOWEVER, it WILL search /usr/local/lib. So that Linux users needn't modify
# their LD_LIBRARY_PATH if installing to /usr/local, we force the LIBDIR to be
# lib/.
# Note: CMake 3.0 fixes this issue. When we move to CMake 3.0, we can
# remove this if-statement. See issue #151.
IF("${CMAKE_INSTALL_PREFIX}" STREQUAL "/usr/local" OR
        "${CMAKE_INSTALL_PREFIX}" STREQUAL "/usr/local/")
    # Overwrite both of these variables; we use both of them.
    SET(CMAKE_INSTALL_LIBDIR "lib")
    SET(CMAKE_INSTALL_FULL_LIBDIR
        "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}")
ENDIF()


IF(NOT MSVC AND NOT XCODE AND NOT CMAKE_BUILD_TYPE)
    SET(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING
        "Debug, RelWithDebInfo (recommended), or Release build"
        FORCE)
ENDIF()


## Choose the maximum level of x86 instruction set that the compiler is
## allowed to use.
## Was using sse2 but changed to let the compilers choose. Most will
## probably use sse2 or later by default.
## On 64 bit MSVC 2013, the default is sse2 and the argument
## isn't recognized so don't specify it.
if (CMAKE_CL_64)
    set(default_build_inst_set)
else()
    set(default_build_inst_set)
endif()

## This can be set to a different value by the person running CMake.
SET(BUILD_INST_SET ""
    CACHE STRING
    "CPU instruction level compiler is permitted to use (default: let compiler decide).")
MARK_AS_ADVANCED( BUILD_INST_SET )

if (BUILD_INST_SET)
    set(inst_set_to_use ${BUILD_INST_SET})
else()
    set(inst_set_to_use ${default_build_inst_set})
endif()

## When building in any of the Release modes, tell gcc/clang to use
## not-quite most agressive optimization.  Here we
## are specifying *all* of the Release flags, overriding CMake's defaults.
## Watch out for optimizer bugs in particular gcc versions!

IF(${CMAKE_CXX_COMPILER_ID} MATCHES "GNU" OR
   ${CMAKE_CXX_COMPILER_ID} MATCHES "Clang")

    # C++11
    # -----
    # Using C++11 on OSX requires using libc++ instead of libstd++.
    # libc++ is an implementation of the C++ standard library for OSX.
    IF(APPLE)
        IF(XCODE)
            SET(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LANGUAGE_STANDARD "c++11")
            SET(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LIBRARY "libc++")
        ELSE()
            SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
            IF(${CMAKE_CXX_COMPILER_ID} MATCHES "Clang")
                SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
            ENDIF()
        ENDIF()
    ELSE() # not APPLE
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    ENDIF()

    if (inst_set_to_use)
        string(TOLOWER ${inst_set_to_use} GCC_INST_SET)
        set(GCC_INST_SET "-m${GCC_INST_SET}")
    else()
        set(GCC_INST_SET)
    endif()

    # Get the gcc or clang version number in major.minor.build format
    execute_process(COMMAND ${CMAKE_C_COMPILER} -dumpversion
                    OUTPUT_VARIABLE GCC_VERSION)

    # Unrolling fixed-count loops was a useful optimization for Simmatrix
    # in earlier gcc versions.
    # Doesn't have a big effect for current compiler crop and may be
    # pushing our luck with optimizer bugs. So let the compilers decide
    # how to handle loops instead.
    ##SET(GCC_OPT_ENABLE "-funroll-loops")

    # If you know of optimization bugs that affect Simbody in particular
    # gcc versions, this is the place to turn off those optimizations.
    SET(GCC_OPT_DISABLE)
    # We know Gcc 4.4.3 on Ubuntu 10 is buggy.
    IF(${CMAKE_CXX_COMPILER_ID} MATCHES "GNU")
      if (GCC_VERSION VERSION_EQUAL 4.4)
        SET(GCC_OPT_DISABLE
        "-fno-strict-aliasing -fno-tree-vrp -fno-guess-branch-probability")
      endif()
    ENDIF()

    # C++
    SET(BUILD_CXX_FLAGS_DEBUG          "-g ${GCC_INST_SET}")
    SET(BUILD_CXX_FLAGS_RELEASE
      "-DNDEBUG -O2 ${GCC_OPT_ENABLE} ${GCC_OPT_DISABLE} ${GCC_INST_SET}")
    SET(BUILD_CXX_FLAGS_RELWITHDEBINFO
    "-DNDEBUG -O2 -g ${GCC_OPT_ENABLE} ${GCC_OPT_DISABLE} ${GCC_INST_SET}")
    SET(BUILD_CXX_FLAGS_MINSIZEREL     "-DNDEBUG -Os ${GCC_INST_SET}")

    # C
    SET(BUILD_C_FLAGS_DEBUG            "-g ${GCC_INST_SET}")
    SET(BUILD_C_FLAGS_RELEASE
      "-DNDEBUG -O2 ${GCC_OPT_ENABLE} ${GCC_OPT_DISABLE} ${GCC_INST_SET}")
    SET(BUILD_C_FLAGS_RELWITHDEBINFO
    "-DNDEBUG -O2 -g ${GCC_OPT_ENABLE} ${GCC_OPT_DISABLE} ${GCC_INST_SET}")
    SET(BUILD_C_FLAGS_MINSIZEREL       "-DNDEBUG -Os ${GCC_INST_SET}")

    # C++
    SET(CMAKE_CXX_FLAGS_DEBUG ${BUILD_CXX_FLAGS_DEBUG}
        CACHE STRING "Can't change here -- see BUILD_CXX..." FORCE)
    SET(CMAKE_CXX_FLAGS_RELEASE ${BUILD_CXX_FLAGS_RELEASE}
        CACHE STRING "Can't change here -- see BUILD_CXX..." FORCE)
    SET(CMAKE_CXX_FLAGS_RELWITHDEBINFO ${BUILD_CXX_FLAGS_RELWITHDEBINFO}
        CACHE STRING "Can't change here -- see BUILD_CXX..." FORCE)
    SET(CMAKE_CXX_FLAGS_MINSIZEREL ${BUILD_CXX_FLAGS_MINSIZEREL}
        CACHE STRING "Can't change here -- see BUILD_CXX..." FORCE)

    # C
    SET(CMAKE_C_FLAGS_DEBUG ${BUILD_C_FLAGS_DEBUG}
        CACHE STRING "Can't change here -- see BUILD_C..." FORCE)
    SET(CMAKE_C_FLAGS_RELEASE ${BUILD_C_FLAGS_RELEASE}
        CACHE STRING "Can't change here -- see BUILD_C..." FORCE)
    SET(CMAKE_C_FLAGS_RELWITHDEBINFO ${BUILD_C_FLAGS_RELWITHDEBINFO}
        CACHE STRING "Can't change here -- see BUILD_C..." FORCE)
    SET(CMAKE_C_FLAGS_MINSIZEREL ${BUILD_C_FLAGS_MINSIZEREL}
        CACHE STRING "Can't change here -- see BUILD_C..." FORCE)

ENDIF()

## When building in any of the Release modes, tell VC++ cl compiler to use
## intrinsics (i.e. sqrt instruction rather than sqrt subroutine) with
## flag /Oi.
## Caution: can't use CMAKE_CXX_COMPILER_ID MATCHES MSVC here because
## "MSVC" is a predefined CMAKE variable and will get expanded to 1 or 0
IF(MSVC)
    if (inst_set_to_use)
        string(TOUPPER ${inst_set_to_use} CL_INST_SET)
        set(CL_INST_SET "/arch:${CL_INST_SET}")
    else()
        set(CL_INST_SET)
    endif()

    set(BUILD_LIMIT_PARALLEL_COMPILES "" CACHE STRING
        "Set a maximum number of simultaneous compilations.")
    mark_as_advanced(BUILD_LIMIT_PARALLEL_COMPILES)
    set(mxcpu ${BUILD_LIMIT_PARALLEL_COMPILES}) # abbreviation

    ## C++
    SET(BUILD_CXX_FLAGS_DEBUG
    "/D _DEBUG /MDd /Od /Ob0 /RTC1 /Zi /GS- ${CL_INST_SET}")
    SET(BUILD_CXX_FLAGS_RELEASE
    "/D NDEBUG /MD  /O2 /Ob2 /Oi /GS- ${CL_INST_SET}")
    SET(BUILD_CXX_FLAGS_RELWITHDEBINFO
    "/D NDEBUG /MD  /O2 /Ob2 /Oi /Zi /GS- ${CL_INST_SET}")
    SET(BUILD_CXX_FLAGS_MINSIZEREL
    "/D NDEBUG /MD  /O1 /Ob1 /Oi /GS- ${CL_INST_SET}")

    ## C
    SET(BUILD_C_FLAGS_DEBUG
    "/D _DEBUG /MDd /Od /Ob0 /RTC1 /Zi /GS- ${CL_INST_SET}")
    SET(BUILD_C_FLAGS_RELEASE
    "/D NDEBUG /MD  /O2 /Ob2 /Oi /GS- ${CL_INST_SET}")
    SET(BUILD_C_FLAGS_RELWITHDEBINFO
    "/D NDEBUG /MD  /O2 /Ob2 /Oi /Zi /GS- ${CL_INST_SET}")
    SET(BUILD_C_FLAGS_MINSIZEREL
    "/D NDEBUG /MD  /O1 /Ob1 /Oi /GS- ${CL_INST_SET}")

    ## C++
    SET(CMAKE_CXX_FLAGS_DEBUG "/MP${mxcpu} ${BUILD_CXX_FLAGS_DEBUG}"
        CACHE STRING "Can't change here -- see BUILD_CXX..." FORCE)
    SET(CMAKE_CXX_FLAGS_RELEASE "/MP${mxcpu} ${BUILD_CXX_FLAGS_RELEASE}"
        CACHE STRING "Can't change here -- see BUILD_CXX..." FORCE)
    SET(CMAKE_CXX_FLAGS_RELWITHDEBINFO "/MP${mxcpu} ${BUILD_CXX_FLAGS_RELWITHDEBINFO}"
        CACHE STRING "Can't change here -- see BUILD_CXX..." FORCE)
    SET(CMAKE_CXX_FLAGS_MINSIZEREL "/MP${mxcpu} ${BUILD_CXX_FLAGS_MINSIZEREL}"
        CACHE STRING "Can't change here -- see BUILD_CXX..." FORCE)

    ## C
    SET(CMAKE_C_FLAGS_DEBUG "/MP${mxcpu} ${BUILD_C_FLAGS_DEBUG}"
        CACHE STRING "Can't change here -- see BUILD_C_..." FORCE)
    SET(CMAKE_C_FLAGS_RELEASE "/MP${mxcpu} ${BUILD_C_FLAGS_RELEASE}"
        CACHE STRING "Can't change here -- see BUILD_C_..." FORCE)
    SET(CMAKE_C_FLAGS_RELWITHDEBINFO "/MP${mxcpu} ${BUILD_C_FLAGS_RELWITHDEBINFO}"
        CACHE STRING "Can't change here -- see BUILD_C_..." FORCE)
    SET(CMAKE_C_FLAGS_MINSIZEREL "/MP${mxcpu} ${BUILD_C_FLAGS_MINSIZEREL}"
        CACHE STRING "Can't change here -- see BUILD_C_..." FORCE)

ENDIF()

# Collect up information about the version of the simbody library we're building
# and make it available to the code so it can be built into the binaries.
# TODO removed SVN_REVSION; replace with GIT_SHA1
# http://stackoverflow.com/questions/1435953/how-can-i-pass-git-sha1-to-compiler-as-definition-using-cmake

# CMake quotes automatically when building Visual Studio projects but we need
# to add them ourselves for Linux or Cygwin. Two cases to avoid duplicate quotes
# in Visual Studio which end up in the binary.

IF (${CMAKE_GENERATOR} MATCHES "Visual Studio")
   SET(NEED_QUOTES FALSE)
ELSE (${CMAKE_GENERATOR} MATCHES "Visual Studio")
   SET(NEED_QUOTES TRUE)
ENDIF (${CMAKE_GENERATOR} MATCHES "Visual Studio")

##TODO: doesn't work without quotes in nightly build
SET(NEED_QUOTES TRUE)

IF(NEED_QUOTES)
   ADD_DEFINITIONS(-DSimTK_SIMBODY_COPYRIGHT_YEARS="${SIMBODY_COPYRIGHT_YEARS}"
                   -DSimTK_SIMBODY_AUTHORS="${SIMBODY_AUTHORS}")
ELSE(NEED_QUOTES)
   ADD_DEFINITIONS(-DSimTK_SIMBODY_COPYRIGHT_YEARS=${SIMBODY_COPYRIGHT_YEARS}
                   -DSimTK_SIMBODY_AUTHORS=${SIMBODY_AUTHORS})
ENDIF(NEED_QUOTES)


# Determine which math libraries to use for this platform.
# Intel MKL: mkl_intel_c_dll;mkl_sequential_dll;mkl_core_dll
SET(BUILD_USING_OTHER_LAPACK "" CACHE STRING
 "If you have your own Lapack and Blas, put library basenames here, separated by semicolons. See LAPACK_BEING_USED to see what's actually being used.")
if(WIN32)
    set(LAPACK_PLATFORM_DEFAULT liblapack;libblas)
else()
    find_package(BLAS)
    find_package(LAPACK)
    if(BLAS_FOUND AND LAPACK_FOUND)
        set(LAPACK_PLATFORM_DEFAULT ${BLAS_LIBRARIES} ${LAPACK_LIBRARIES})
    else()
        message(WARNING "Could not find blas/lapack")
    endif()
endif()
SET(LAPACK_BEING_USED ${LAPACK_PLATFORM_DEFAULT} CACHE STRING
    "Basename of the actual Lapack library we're depending on; can't change here; see variable BUILD_USING_OTHER_LAPACK." FORCE)

IF(BUILD_USING_OTHER_LAPACK)
    SET(LAPACK_BEING_USED ${BUILD_USING_OTHER_LAPACK} CACHE STRING
"Basename of the actual Lapack library we're depending on; can't change here; see variable BUILD_USING_OTHER_LAPACK." FORCE)
ENDIF(BUILD_USING_OTHER_LAPACK)

IF(UNIX)
    IF(NOT APPLE)
        SET(REALTIME_LIB rt)
    ENDIF()
    SET(MATH_LIBRARY m)
    SET(DL_LIBRARY dl)

    # use CMAKE_THREAD_LIBS_INIT var from findthreads.cmake
    SET(CMAKE_THREAD_LIBS_INIT pthread)
ENDIF()

IF(MSVC)
    if (${PLATFORM_ABI} MATCHES "x64")
        SET(CMAKE_THREAD_LIBS_INIT pthreadVC2_x64)
    else()
        SET(CMAKE_THREAD_LIBS_INIT pthreadVC2)
    endif()
ENDIF()

SET(MATH_LIBS_TO_USE    ${LAPACK_BEING_USED} ${CMAKE_THREAD_LIBS_INIT} ${REALTIME_LIB} ${DL_LIBRARY} ${MATH_LIBRARY})
SET(MATH_LIBS_TO_USE_VN ${MATH_LIBS_TO_USE})
#
# Allow automated build and dashboard.
#
INCLUDE (Dart)
## When in Debug mode and running valgrind, some of the test
## cases take longer than the default 1500 seconds.
SET(DART_TESTING_TIMEOUT 7200)

IF (BUILD_TESTING)

    #IF (UNIX AND NOT CYGWIN AND NOT APPLE)
    #  IF (NOT CMAKE_BUILD_TYPE OR CMAKE_BUILD_TYPE MATCHES Debug)
    #    ADD_DEFINITIONS(-fprofile-arcs -ftest-coverage)
    #    LINK_LIBRARIES(gcov)
    #  ENDIF (NOT CMAKE_BUILD_TYPE OR CMAKE_BUILD_TYPE MATCHES Debug)
    #ENDIF (UNIX AND NOT CYGWIN AND NOT APPLE)

    #
    # Testing
    #
    ENABLE_TESTING()

    # Make a RUN_TESTS_PARALLEL target (thanks, Kevin!)
    # Specify number of cores to run for testing
    SET(TESTING_PROCESSOR_COUNT 4 CACHE STRING
        "Number of CPUs to be used by the RUN_TESTS_PARALLEL target.")
    MARK_AS_ADVANCED(TESTING_PROCESSOR_COUNT)
    SET (cmd ${CMAKE_CTEST_COMMAND} -j${TESTING_PROCESSOR_COUNT})
    IF(MSVC OR XCODE)
        SET (cmd ${cmd} -C ${CMAKE_CFG_INTDIR})
    ELSE()
        SET (cmd ${cmd} -C ${CMAKE_BUILD_TYPE})
    ENDIF()
    ADD_CUSTOM_TARGET (RUN_TESTS_PARALLEL
        COMMAND ${cmd}
    )

ENDIF (BUILD_TESTING)

INCLUDE(ApiDoxygen.cmake)

# Each build should look in our local binary directory to find the
# earlier-built libraries that it depends on.
LINK_DIRECTORIES(${BUILD_BINARY_DIR})

# Specify where visualizer should be installed. This needs to be in the
# root CMakeLists.txt so the cmake config file can see this value.
#
# Also specify where include files are installed.
IF(WIN32)
  # Install visualizer to bin, since it needs to be co-located with dll's
  SET(SIMBODY_VISUALIZER_REL_INSTALL_DIR ${CMAKE_INSTALL_BINDIR})
  # Install include files into base include folder since it's a sandbox
  SET(SIMBODY_INCLUDE_INSTALL_DIR ${CMAKE_INSTALL_INCLUDEDIR})
ELSE()
  # Visualizer is not intended to be a user executable. Proper place is
  # inside the lib directory
  SET(SIMBODY_VISUALIZER_REL_INSTALL_DIR ${CMAKE_INSTALL_LIBEXECDIR}/simbody)
  # Install include files in simbody subfolder to avoid polluting the
  # global build folder
  SET(SIMBODY_INCLUDE_INSTALL_DIR ${CMAKE_INSTALL_INCLUDEDIR}/simbody)
ENDIF()
SET(SIMBODY_VISUALIZER_INSTALL_DIR
    ${CMAKE_INSTALL_PREFIX}/${SIMBODY_VISUALIZER_REL_INSTALL_DIR})

# Each of these returns a list of API include directories for
# use by the later builds.
ADD_CUSTOM_TARGET(PlatformFiles ALL)
ADD_SUBDIRECTORY( Platform )
# PLATFORM_INCLUDE_DIRECTORIES now set
ADD_SUBDIRECTORY( SimTKcommon )
# SimTKCOMMON_INCLUDE_DIRECTORIES now set
ADD_SUBDIRECTORY( SimTKmath )
# SimTKMATH_INCLUDE_DIRECTORIES now set
ADD_SUBDIRECTORY( Simbody )
# SimTKSIMBODY_INCLUDE_DIRECTORIES now set (but not used)

IF( BUILD_EXAMPLES )
  ADD_SUBDIRECTORY( examples )
ENDIF( BUILD_EXAMPLES )

FILE(GLOB TOPLEVEL_DOCS LICENSE.txt *.md doc/*.pdf doc/*.txt doc/*.md)
INSTALL(FILES ${TOPLEVEL_DOCS} DESTINATION ${CMAKE_INSTALL_DOCDIR})

# Add uninstall target
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/cmake/cmake_uninstall.cmake"
  IMMEDIATE @ONLY)
add_custom_target(uninstall
  "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake/cmake_uninstall.cmake")

# Make the cmake config files
set(PKG_NAME ${PROJECT_NAME})
set(PKG_LIBRARIES
    ${SimTKSIMBODY_LIBRARY_NAME}
    ${SimTKMATH_LIBRARY_NAME}
    ${SimTKCOMMON_LIBRARY_NAME}
    )

if (WIN32)
  set(SIMBODY_CMAKE_DIR cmake)
elseif (UNIX)
  set(SIMBODY_CMAKE_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/simbody/)
endif ()

# Configure SimbodyConfig.cmake in a way that allows the installation to be
# relocatable.
INCLUDE(CMakePackageConfigHelpers)
CONFIGURE_PACKAGE_CONFIG_FILE(
    ${CMAKE_SOURCE_DIR}/cmake/SimbodyConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/cmake/SimbodyConfigForInstall.cmake
    INSTALL_DESTINATION "${SIMBODY_CMAKE_DIR}"
    PATH_VARS # Variables to edit in the SimbodyConfig.cmake.in.
        CMAKE_INSTALL_PREFIX
        SIMBODY_INCLUDE_INSTALL_DIR
        CMAKE_INSTALL_LIBDIR
        SIMBODY_VISUALIZER_INSTALL_DIR
        SIMBODY_INSTALL_DOXYGENDIR
    )
INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/cmake/SimbodyConfigForInstall.cmake
    DESTINATION ${SIMBODY_CMAKE_DIR}
    RENAME SimbodyConfig.cmake)

# Create a file that allows clients to Simbody to ensure they have the version
# of Simbody they want.
# Requires CMake 2.8.8.
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/cmake/SimbodyConfigVersion.cmake
    VERSION "${SIMBODY_VERSION}"
    COMPATIBILITY SameMajorVersion)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/cmake/SimbodyConfigVersion.cmake
        DESTINATION ${SIMBODY_CMAKE_DIR})

install(EXPORT SimbodyTargets DESTINATION
  "${SIMBODY_CMAKE_DIR}") #optionally: COMPONENT dev

# Install a sample CMakeLists.txt that uses SimbodyConfig.cmake.
INSTALL(FILES ${CMAKE_SOURCE_DIR}/cmake/SampleCMakeLists.txt
    DESTINATION ${SIMBODY_CMAKE_DIR})

# Make the pkgconfig file: select the proper flags depending on the platform
IF (WIN32)
    IF( ${CMAKE_SIZEOF_VOID_P} EQUAL 8)
        # win 64 bits
    SET(PKGCONFIG_PLATFORM_LIBS "-lliblapack -llibblas -lpthreadVC2_x64")
    ELSE()
    SET(PKGCONFIG_PLATFORM_LIBS "-lliblapack -llibblas -lpthreadVC2")
    ENDIF()
ELSEIF (APPLE)
    SET(PKGCONFIG_PLATFORM_LIBS "-llapack -lblas -lpthread -ldl")
ELSE()
    SET(PKGCONFIG_PLATFORM_LIBS "-llapack -lblas -lpthread -lrt -ldl -lm")
ENDIF()

CONFIGURE_FILE(${CMAKE_SOURCE_DIR}/cmake/pkgconfig/simbody.pc.in
    ${CMAKE_CURRENT_BINARY_DIR}/cmake/pkgconfig/simbody.pc @ONLY)
INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/cmake/pkgconfig/simbody.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig/)
