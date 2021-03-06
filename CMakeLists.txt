# Project setup
###############

cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
if(COMMAND cmake_policy)
  cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)


project(ViennaCL)

set(VERSION_MAJOR 1)
set(VERSION_MINOR 4)
set(VERSION_PATCH 2)
set(VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH})

SET(CMAKE_VERBOSE_MAKEFILE ON)   #uncomment this line to increase CMake verbosity

#SET(BOOSTPATH "C:/Program\ Files\ (x86)/boost/boost_1_42")    #Boost path for other systems - customize as needed

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

# Add visibility of headers
# Necessary for Qt-Creator usage.
################
file( GLOB_RECURSE MAKE_HEADERS_VISIBLE_SRC
    *.hpp *.h)
add_custom_target( MAKE_HEADERS_VISIBLE SOURCES ${MAKE_HEADERS_VISIBLE_SRC} )


# Common options
################

set(VIENNACL_SRC_DIST TRUE)
include(ViennaCLCommon)

# Set release build:
IF(DEFINED CMAKE_BUILD_TYPE)
 SET (CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE})
ELSE()
 SET (CMAKE_BUILD_TYPE Release)
ENDIF()

# User options
##############

option(BUILD_DOXYGEN_DOCS "Build the Doxygen-generated API docs" OFF)

option(BUILD_MANUAL "Build the PDF manual" OFF)

cmake_dependent_option(ENABLE_DIST "Enable the the distribution tarball target" ON "BUILD_DOXYGEN_DOCS;BUILD_MANUAL" OFF)

mark_as_advanced(BUILD_DOXYGEN_DOCS BUILD_MANUAL ENABLE_DIST)

# Find prerequisites
####################

if(BUILD_DOXYGEN_DOCS)
   find_package(Doxygen REQUIRED)
endif()

if(BUILD_MANUAL)
   find_package(LATEX REQUIRED)
endif()

# Compile options
#################

include_directories(BEFORE ${PROJECT_SOURCE_DIR})

IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  INCLUDE_DIRECTORIES("/opt/local/include") 
  set(CMAKE_EXE_LINKER_FLAGS "-framework OpenCL")
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")


# Subdirectories
################

add_subdirectory(auxiliary)

if(BUILD_EXAMPLES)
   add_subdirectory(examples)
endif()

if(BUILD_TESTING)
   INCLUDE(CTest)
   add_subdirectory(tests)
endif()

add_subdirectory(doc)

# Install
#########

install(DIRECTORY viennacl ${CMAKE_CURRENT_BINARY_DIR}/viennacl
   DESTINATION ${INSTALL_INCLUDE_DIR} COMPONENT dev
   FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")

# Dist target
#############

if(ENABLE_DIST)
   file(MAKE_DIRECTORY "${PROJECT_BINARY_DIR}/dist")


   set(DISTDIR "${PROJECT_BINARY_DIR}/dist/ViennaCL-${VERSION}-all-arch")
   configure_file(cmake/CMakeLists.txt.in "${DISTDIR}/CMakeLists.txt" @ONLY)

   set(DIST_SOURCES_DIR "${PROJECT_BINARY_DIR}/dist/ViennaCL-${VERSION}-src")
   configure_file(cmake/CMakeLists.txt.in.src "${DIST_SOURCES_DIR}/CMakeLists.txt" @ONLY)

   #
   # Full package:
   #
   add_custom_target(dist
      COMMAND "${CMAKE_COMMAND}" -E make_directory "${DISTDIR}"

      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/LICENSE" "${DISTDIR}/LICENSE"

      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/changelog" "${DISTDIR}/changelog"

      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/README" "${DISTDIR}/README"

      COMMAND "${CMAKE_COMMAND}" -E make_directory "${DISTDIR}/build"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/build/README" "${DISTDIR}/build/README"

      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/CL" "${DISTDIR}/CL"

      COMMAND "${CMAKE_COMMAND}" -E make_directory "${DISTDIR}/cmake"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/cmake/copymanual.cmake" "${DISTDIR}/cmake/copymanual.cmake"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/cmake/FindMTL.cmake" "${DISTDIR}/cmake/FindMTL.cmake"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/cmake/FindOpenCL.cmake" "${DISTDIR}/cmake/FindOpenCL.cmake"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/cmake/FindViennaProfiler.cmake" "${DISTDIR}/cmake/FindViennaProfiler.cmake"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/cmake/ViennaCLCommon.cmake" "${DISTDIR}/cmake/ViennaCLCommon.cmake"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/cmake/ViennaCLConfig.cmake.in" "${DISTDIR}/cmake/ViennaCLConfig.cmake.in"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/cmake/ViennaCLConfigVersion.cmake.in" "${DISTDIR}/cmake/ViennaCLConfigVersion.cmake.in"

      COMMAND "${CMAKE_COMMAND}" -E make_directory "${DISTDIR}/doc"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/doc/viennacl.pdf" "${DISTDIR}/doc/viennacl.pdf"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${CMAKE_CURRENT_BINARY_DIR}/doc/doxygen"
         "${DISTDIR}/doc/doxygen"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/doc/viennacl.pdf" "${DISTDIR}/doc/viennacl.pdf"

      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/doc/Doxyfile.in" "${DISTDIR}/doc/Doxyfile.in"

      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/examples" "${DISTDIR}/examples"

      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/external" "${DISTDIR}/external"

      COMMAND "${CMAKE_COMMAND}" -E make_directory "${DISTDIR}/tests"
      COMMAND "${CMAKE_COMMAND}" -E make_directory "${DISTDIR}/tests/src"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/tests/src" "${DISTDIR}/tests/src"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/tests/CMakeLists.txt" "${DISTDIR}/tests/CMakeLists.txt"

      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/viennacl" "${DISTDIR}/viennacl"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${CMAKE_CURRENT_BINARY_DIR}/viennacl/linalg/kernels"
         "${DISTDIR}/viennacl/linalg/kernels"

      COMMAND "${CMAKE_COMMAND}" -E tar czf ViennaCL-${VERSION}-all-arch.tar.gz
           ViennaCL-${VERSION}-all-arch
      WORKING_DIRECTORY "${PROJECT_BINARY_DIR}/dist"
      COMMENT "Generating dist/ViennaCL-${VERSION}-all-arch.tar.gz"
      VERBATIM)
   add_dependencies(dist kernels apidoc manual)

   #
   # Sources only (no generated files):
   #
   foreach(f LICENSE 
             README 
             changelog
             cmake/copymanual.cmake
             cmake/FindMTL.cmake
             cmake/FindOpenCL.cmake
             cmake/FindViennaProfiler.cmake 
             cmake/ViennaCLCommon.cmake
             cmake/ViennaCLConfig.cmake.in
             cmake/ViennaCLConfigVersion.cmake.in 
             auxiliary/README
             build/README)
      configure_file(${f} "${DIST_SOURCES_DIR}/${f}" COPYONLY)
   endforeach()

   add_custom_target(copymanual-src ALL
      "${CMAKE_COMMAND}" 
         -DSRC="${PROJECT_SOURCE_DIR}/doc/manual/"
         -DDST="${DIST_SOURCES_DIR}/doc/manual/"
         -P ${CMAKE_SOURCE_DIR}/cmake/copymanual.cmake
   )

   add_custom_target(dist-src
   
      # Set up directory layout:
      COMMAND "${CMAKE_COMMAND}" -E make_directory "${DIST_SOURCES_DIR}"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/compressed_matrix" "${DIST_SOURCES_DIR}/auxiliary/compressed_matrix"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/coordinate_matrix" "${DIST_SOURCES_DIR}/auxiliary/coordinate_matrix"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/ell_matrix" "${DIST_SOURCES_DIR}/auxiliary/ell_matrix"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/hyb_matrix" "${DIST_SOURCES_DIR}/auxiliary/hyb_matrix"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/matrix_col" "${DIST_SOURCES_DIR}/auxiliary/matrix_col"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/matrix_row" "${DIST_SOURCES_DIR}/auxiliary/matrix_row"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/scalar" "${DIST_SOURCES_DIR}/auxiliary/scalar"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/vector" "${DIST_SOURCES_DIR}/auxiliary/vector"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/fft" "${DIST_SOURCES_DIR}/auxiliary/fft"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/svd" "${DIST_SOURCES_DIR}/auxiliary/svd"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/spai" "${DIST_SOURCES_DIR}/auxiliary/spai"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/nmf" "${DIST_SOURCES_DIR}/auxiliary/nmf"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/auxiliary/ilu" "${DIST_SOURCES_DIR}/auxiliary/ilu"
      COMMAND "${CMAKE_COMMAND}" -E copy
         "${PROJECT_SOURCE_DIR}/auxiliary/dump_random" "${DIST_SOURCES_DIR}/auxiliary/random"
      COMMAND "${CMAKE_COMMAND}" -E copy
         "${PROJECT_SOURCE_DIR}/auxiliary/CMakeLists.txt" "${DIST_SOURCES_DIR}/auxiliary/CMakeLists.txt"
      COMMAND "${CMAKE_COMMAND}" -E copy
         "${PROJECT_SOURCE_DIR}/auxiliary/converter.cpp" "${DIST_SOURCES_DIR}/auxiliary/converter.cpp"
      COMMAND "${CMAKE_COMMAND}" -E copy
         "${PROJECT_SOURCE_DIR}/auxiliary/generate-blas3-prod-align1.cpp" "${DIST_SOURCES_DIR}/auxiliary/generate-blas3-prod-align1.cpp"
      COMMAND "${CMAKE_COMMAND}" -E copy
         "${PROJECT_SOURCE_DIR}/auxiliary/generate-blas3-prod16-align1.cpp" "${DIST_SOURCES_DIR}/auxiliary/generate-blas3-prod16-align1.cpp"
      COMMAND "${CMAKE_COMMAND}" -E copy
         "${PROJECT_SOURCE_DIR}/auxiliary/generate-blas3-prod-amd.cpp" "${DIST_SOURCES_DIR}/auxiliary/generate-blas3-prod-amd.cpp"
      COMMAND "${CMAKE_COMMAND}" -E copy
         "${PROJECT_SOURCE_DIR}/auxiliary/generate-blas3-solve-align1.cpp" "${DIST_SOURCES_DIR}/auxiliary/generate-blas3-solve-align1.cpp"

      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/CL" "${DIST_SOURCES_DIR}/CL"

      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/examples" "${DIST_SOURCES_DIR}/examples"

      COMMAND "${CMAKE_COMMAND}" -E make_directory "${DIST_SOURCES_DIR}/tests"
      COMMAND "${CMAKE_COMMAND}" -E make_directory "${DIST_SOURCES_DIR}/tests/src"
      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/tests/src" "${DIST_SOURCES_DIR}/tests/src"
      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/tests/CMakeLists.txt" "${DIST_SOURCES_DIR}/tests/CMakeLists.txt"

      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/external" "${DIST_SOURCES_DIR}/external"

      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/doc/manual/figures" "${DIST_SOURCES_DIR}/doc/manual/figures"

      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/doc/Doxyfile.in" "${DIST_SOURCES_DIR}/doc/Doxyfile.in"

      COMMAND "${CMAKE_COMMAND}" -E copy_if_different
         "${PROJECT_SOURCE_DIR}/doc/CMakeLists.txt" "${DIST_SOURCES_DIR}/doc/CMakeLists.txt"

      COMMAND "${CMAKE_COMMAND}" -E copy_directory
         "${PROJECT_SOURCE_DIR}/viennacl" "${DIST_SOURCES_DIR}/viennacl"

      COMMAND "${CMAKE_COMMAND}" -E remove_directory
         "${DIST_SOURCES_DIR}/viennacl/linalg/kernels"

      # Create archive:
      COMMAND "${CMAKE_COMMAND}" -E tar czf ViennaCL-${VERSION}-src.tar.gz
           ViennaCL-${VERSION}-src
      WORKING_DIRECTORY "${PROJECT_BINARY_DIR}/dist"
      COMMENT "Generating dist/ViennaCL-${VERSION}-src.tar.gz"
      VERBATIM)
   add_dependencies(dist-src copymanual-src)

endif()
