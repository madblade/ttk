# Add compile flags and defintions to the target
# according to the options selected by the user.
#
# Usage:
# ttk_set_compile_options(<library_name>)
#
function(ttk_set_compile_options library)
    target_compile_options(${library} PRIVATE -Wall)

    if(Boost_FOUND)
      target_link_libraries(${library} PUBLIC Boost::boost)
    endif()

    if(Boost_LIBRARIES)
      target_link_libraries(${library} PUBLIC Boost::system)
    endif()

    if (TTK_ENABLE_KAMIKAZE)
        target_compile_definitions(${library} PUBLIC TTK_ENABLE_KAMIKAZE)
    endif()

  if(NOT MSVC)
    if (TTK_ENABLE_CPU_OPTIMIZATION)
      target_compile_options(${library}
        PRIVATE $<$<CONFIG:Release>:-march=native -O3 -Wfatal-errors>)
    endif()

    target_compile_options(${library} PRIVATE $<$<CONFIG:Debug>:-O0 -g -pg>)
  endif()
  if(MSVC)
    # disable warnings
    target_compile_options(${library} PUBLIC /wd4005 /wd4061 /wd4100 /wd4146 /wd4221 /wd4242 /wd4244 /wd4245 /wd4263 /wd4264 /wd4267 /wd4273 /wd4275 /wd4296 /wd4305 /wd4365 /wd4371 /wd4435 /wd4456 /wd4457 /wd4514 /wd4619 /wd4625 /wd4626 /wd4628 /wd4668 /wd4701 /wd4702 /wd4710 /wd4800 /wd4820 /wd4996 /wd5027 /wd5029 /wd5031)
  endif()

    if (TTK_ENABLE_OPENMP)
        target_compile_definitions(${library} PUBLIC TTK_ENABLE_OPENMP)
        target_compile_options(${library} PUBLIC ${OpenMP_CXX_FLAGS})
        target_link_libraries(${library} PUBLIC ${OpenMP_CXX_LIBRARIES})
    endif()

    if (TTK_ENABLE_MPI)
        target_compile_definitions(${library} PUBLIC TTK_ENABLE_MPI)
        target_include_directories(${library} PUBLIC ${MPI_CXX_INCLUDE_PATH})
        target_link_libraries(${library} PUBLIC ${MPI_CXX_LIBRARIES})
    endif()

    if (TTK_ENABLE_ZFP)
        target_compile_definitions(${library} PUBLIC TTK_ENABLE_ZFP)
        target_link_libraries(${library} PUBLIC zfp::zfp)
    endif()

    if (TTK_ENABLE_ZLIB)
        target_compile_definitions(${library} PUBLIC TTK_ENABLE_ZLIB)
        target_include_directories(${library} PUBLIC ${ZLIB_INCLUDE_DIR})
        target_link_libraries(${library} PUBLIC ${ZLIB_LIBRARY})
    endif()

    if (TTK_ENABLE_SQLITE3)
        target_compile_definitions(${library} PUBLIC TTK_ENABLE_SQLITE3)
        target_include_directories(${library} PUBLIC ${SQLITE3_INCLUDE_DIR})
        target_link_libraries(${library} PUBLIC ${SQLITE3_LIBRARY})
    endif()

    if (TTK_ENABLE_64BIT_IDS)
        target_compile_definitions(${library} PUBLIC TTK_ENABLE_64BIT_IDS)
    endif()

    if (PYTHON_NUMPY_INCLUDE_DIR)
    target_compile_definitions(${library} PUBLIC TTK_PYTHON_FOUND)
    endif()

endfunction()

function(ttk_print_summary)
    message(STATUS "ttk options -----------------------------------------------------------------")
    message(STATUS "TTK_ENABLE_CPU_OPTIMIZATION: ${TTK_ENABLE_CPU_OPTIMIZATION}")
    message(STATUS "TTK_ENABLE_KAMIKAZE: ${TTK_ENABLE_KAMIKAZE}")
    message(STATUS "TTK_ENABLE_OPENMP: ${TTK_ENABLE_OPENMP}")
    message(STATUS "TTK_ENABLE_MPI: ${TTK_ENABLE_MPI}")
    message(STATUS "TTK_ENABLE_SQLITE3: ${TTK_ENABLE_SQLITE3}")
    message(STATUS "TTK_ENABLE_ZFP: ${TTK_ENABLE_ZFP}")
    message(STATUS "TTK_ENABLE_ZLIB: ${TTK_ENABLE_ZLIB}")
    message(STATUS "TTK_ENABLE_64BIT_IDS: ${TTK_ENABLE_64BIT_IDS}")
    message(STATUS "ttk build -------------------------------------------------------------------")
    message(STATUS "BOOST_INCLUDE_DIR: ${Boost_INCLUDE_DIR}")

    if(PYTHON_NUMPY_INCLUDE_DIR)
        message(STATUS "Numpy headers: ${PYTHON_NUMPY_INCLUDE_DIR}")
    else()
        message(STATUS "Numpy headers: NOT-FOUND")
    endif()

    if(TTK_PYTHON_MAJOR_VERSION)
      message(STATUS
        "Python version: ${TTK_PYTHON_MAJOR_VERSION}.${TTK_PYTHON_MINOR_VERSION}")
    else()
      message(STATUS
        "Python version: NOT-FOUND")
    endif()
    message(STATUS "CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")
    message(STATUS "TTK_BUILD_DOCUMENTATION: ${TTK_BUILD_DOCUMENTATION}")
    if(TTK_BUILD_DOCUMENTATION)
        message(STATUS "  DOXYGEN_EXECUTABLE: ${DOXYGEN_EXECUTABLE}")
    endif()
    message(STATUS "TTK_BUILD_PARAVIEW_PLUGINS: ${TTK_BUILD_PARAVIEW_PLUGINS}")
    if(TTK_BUILD_PARAVIEW_PLUGINS)
        message(STATUS "  ParaView_DIR: ${ParaView_DIR}")
    endif()
    message(STATUS "TTK_BUILD_STANDALONE_APPS: ${TTK_BUILD_STANDALONE_APPS}")
    message(STATUS "TTK_BUILD_VTK_WRAPPERS: ${TTK_BUILD_VTK_WRAPPERS}")
    if(TTK_BUILD_VTK_WRAPPERS)
        message(STATUS "  VTK_DIR: ${VTK_DIR}")
    endif()
    message(STATUS "ttk install -----------------------------------------------------------------")
    message(STATUS "CMAKE_INSTALL_PREFIX: ${CMAKE_INSTALL_PREFIX}")
    message(STATUS "TTK_INSTALL_PLUGIN_DIR: ${TTK_INSTALL_PLUGIN_DIR}")
    message(STATUS "-----------------------------------------------------------------------------")

endfunction()

function(ttk_find_python)
  find_package(PythonLibs QUIET)
  if(PYTHON_INCLUDE_DIRS)
    include_directories(${PYTHON_INCLUDE_DIRS})

#     if (${CMAKE_VERSION} VERSION_GREATER "3.12"
#       OR ${CMAKE_VERSION} VERSION_EQUAL "3.12")
#       string(REPLACE \".\" \" \"
#         PYTHON_VERSION_LIST ${PYTHONLIBS_VERSION_STRING})
#     else()
      string(REPLACE "." " "
        PYTHON_VERSION_LIST ${PYTHONLIBS_VERSION_STRING})
#     endif()
    separate_arguments(PYTHON_VERSION_LIST)
    list(GET PYTHON_VERSION_LIST 0 PYTHON_MAJOR_VERSION)
    list(GET PYTHON_VERSION_LIST 1 PYTHON_MINOR_VERSION)

    set(TTK_PYTHON_MAJOR_VERSION "${PYTHON_MAJOR_VERSION}"
      CACHE INTERNAL "TTK_PYTHON_MAJOR_VERSION")
    set(TTK_PYTHON_MINOR_VERSION "${PYTHON_MINOR_VERSION}"
      CACHE INTERNAL "TTK_PYTHON_MINOR_VERSION")

    find_path(PYTHON_NUMPY_INCLUDE_DIR numpy/arrayobject.h PATHS
      ${PYTHON_INCLUDE_DIRS}
      /usr/lib/python${PYTHON_MAJOR_VERSION}.${PYTHON_MINOR_VERSION}/site-packages/numpy/core/include/
      /usr/local/lib/python${PYTHON_MAJOR_VERSION}.${PYTHON_MINOR_VERSION}/site-packages/numpy/core/include)
    if(PYTHON_NUMPY_INCLUDE_DIR)
      include_directories(${PYTHON_NUMPY_INCLUDE_DIR})
    endif()
  endif()
endfunction()

set(CMAKE_CXX_STANDARD 11)

# Set a predefined build type
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
   message(STATUS "Setting build type to 'Release'.")
   set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
   set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release")
endif()

option(TTK_ENABLE_64BIT_IDS "Enable processing on large datasets" OFF)

option(TTK_ENABLE_KAMIKAZE "Enable Kamikaze compilation mode" OFF)

option(TTK_ENABLE_CPU_OPTIMIZATION "Enable native CPU optimizations" ON)


option(TTK_BUILD_DOCUMENTATION "Build doxygen developer documentation" OFF)
if(TTK_BUILD_DOCUMENTATION)
    find_package(Doxygen)
    if (DOXYGEN_FOUND)
        configure_file(${CMAKE_CURRENT_SOURCE_DIR}/ttk.doxygen
            ${CMAKE_CURRENT_BINARY_DIR}/ttk.doxygen)
        add_custom_target(doc ALL
            ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/ttk.doxygen
            WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
            COMMENT "Generating API documentation with Doxygen" VERBATIM)
        install(DIRECTORY
            ${CMAKE_CURRENT_BINARY_DIR}/doc/html DESTINATION
            ${CMAKE_INSTALL_PREFIX}/share/doc/ttk)
        install(DIRECTORY
          ${CMAKE_SOURCE_DIR}/doc/img DESTINATION
            ${CMAKE_INSTALL_PREFIX}/share/doc/ttk)
    endif()
endif()

find_package(Boost COMPONENTS system)
if(NOT Boost_FOUND)
  find_package(Boost REQUIRED)
endif()

find_package(ZLIB)
if(NOT ZLIB_INCLUDE_DIR)
  option(TTK_ENABLE_ZLIB "Enable Zlib support" OFF)
  message(STATUS "Zlib not found, disabling Zlib support in TTK.")
else()
  option(TTK_ENABLE_ZLIB "Enable Zlib support" ON)
endif()

# TODO: This should be in its own findpackage.cmake file!
# START_FINDSQLITE3
find_path(SQLITE3_INCLUDE_DIR sqlite3.h
    PATHS
    /usr/include
    /usr/local/include)
set(SQLITE3_NAMES ${SQLITE3_NAMES} sqlite3)
find_library(SQLITE3_LIBRARY
    NAMES ${SQLITE3_NAMES}
    PATHS $ENV{SQLITE3_ROOT_DIR}/lib /opt/sqlite3/lib)
if (SQLITE3_LIBRARY AND SQLITE3_INCLUDE_DIR)
  set(SQLITE3_LIBRARIES ${SQLITE3_LIBRARY})
  set(SQLITE3_FOUND "YES")
else (SQLITE3_LIBRARY AND SQLITE3_INCLUDE_DIR)
  set(SQLITE3_FOUND "NO")
endif (SQLITE3_LIBRARY AND SQLITE3_INCLUDE_DIR)
if (SQLITE3_FOUND)
  if (NOT SQLITE3_FIND_QUIETLY)
    message(STATUS "Found SQLITE3: ${SQLITE3_LIBRARIES}")
  endif (NOT SQLITE3_FIND_QUIETLY)
else (SQLITE3_FOUND)
  if (SQLITE3_FIND_REQUIRED)
    message(FATAL_ERROR "Could not find SQLITE3 library...")
  endif (SQLITE3_FIND_REQUIRED)
endif (SQLITE3_FOUND)
mark_as_advanced(
    SQLITE3_LIBRARY
    SQLITE3_INCLUDE_DIR)
# END_FINDSQLITE3
if (SQLITE3_FOUND)
  option(TTK_ENABLE_SQLITE3 "Enable SQLITE3 support" ON)
else()
  option(TTK_ENABLE_SQLITE3 "Enable SQLITE3 support" OFF)
  message(STATUS "SQLITE3 not found, disabling SQLITE3 support in TTK.")
endif()

find_package(ZFP)
if(NOT ZFP_INCLUDE_DIRS)
  option(TTK_ENABLE_ZFP "Enable ZFP support" OFF)
  message(STATUS "ZFP not found, disabling ZFP support in TTK.")
else()
  option(TTK_ENABLE_ZFP "Enable ZFP support" ON)
endif()

if (NOT APPLE)
    if (MSVC)
          option(TTK_ENABLE_OPENMP "Enable OpenMP support" FALSE)
    else()
          option(TTK_ENABLE_OPENMP "Enable OpenMP support" TRUE)
    endif()
endif()
option(TTK_ENABLE_MPI "Enable MPI support" FALSE)

if (TTK_ENABLE_OPENMP)
    find_package(OpenMP REQUIRED)
endif()

if (TTK_ENABLE_MPI)
    find_package(MPI REQUIRED)
endif()

add_subdirectory(base)
if (TTK_BUILD_VTK_WRAPPERS)
    add_subdirectory(vtk)
endif()
