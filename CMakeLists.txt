# - CmakeLists.txt for building an application

#----------------------------------------------------------------------------
# Setup the project
cmake_minimum_required(VERSION 2.6 FATAL_ERROR)
project(padmpi)

#------------------------------------------------------------------------------
# check MPI package...
find_package(MPI REQUIRED)
# modify these var. if needed
set(CMAKE_CXX_COMPILER mpicxx)
#set(CMAKE_CXX_INCLUDE_PATH )

#----------------------------------------------------------------------------
# Find Geant4 package, activating all available UI and Vis drivers by default
# You can set WITH_GEANT4_UIVIS to OFF via the command line or ccmake/cmake-gui
# to build a batch mode only executable
option(WITH_GEANT4_UIVIS "Build example with Geant4 UI and Vis drivers" ON)
if(WITH_GEANT4_UIVIS)
  find_package(Geant4 REQUIRED ui_all vis_all)
else()
  find_package(Geant4 REQUIRED)
endif()

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "RelWithDebInfo")
endif()

#----------------------------------------------------------------------------
# Setup Geant4 include directories and compile definitions
#
include(${Geant4_USE_FILE})

#----------------------------------------------------------------------------
# Find ROOT (required package)
#
find_package(ROOT QUIET)
if(NOT ROOT_FOUND)
  message(STATUS "G4 Examples: ROOT package not found. --> padmpi example disabled")  
  return()
endif()

#----------------------------------------------------------------------------
# Add G4mpi library
#
SET(Geant4_LIBRARIES ${Geant4_LIBRARIES} G4mpi)
ADD_LIBRARY(G4mpi SHARED IMPORTED)
SET_PROPERTY(TARGET G4mpi APPEND PROPERTY IMPORTED_CONFIGURATIONS RELEASE)
SET_TARGET_PROPERTIES(G4mpi PROPERTIES
  IMPORTED_LOCATION_RELEASE "${Geant4_DIR}/../libG4mpi.so"
  IMPORTED_SONAME_RELEASE "libG4mpi.so"
)

#----------------------------------------------------------------------------
# Locate sources and headers for this project
#
#include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB sources ${PROJECT_SOURCE_DIR}/src/*.cc)
file(GLOB headers ${PROJECT_SOURCE_DIR}/include/*.hh)


include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include 
                    ${Geant4_INCLUDE_DIR}
                    ${ROOT_INCLUDE_DIR})


#----------------------------------------------------------------------------
# Add the executable, and link it to the Geant4 libraries
#
add_executable(padmpi padmpi.cc ${sources} ${headers})
target_link_libraries(padmpi ${Geant4_LIBRARIES} ${ROOT_LIBRARIES})
set_target_properties(padmpi
                      PROPERTIES INSTALL_RPATH
                      ${Geant4_DIR}/..
                      BUILD_WITH_INSTALL_RPATH TRUE)

#----------------------------------------------------------------------------
# Copy all scripts to the build directory, i.e. the directory in which we
# build padmpi. This is so that we can run the executable directly because it
# relies on these scripts being in the current working directory.
#
set(padmpi_SCRIPTS
    v.mac
    init.mac
  )

foreach(_script ${padmpi_SCRIPTS})
  configure_file(
    ${PROJECT_SOURCE_DIR}/${_script}
    ${PROJECT_BINARY_DIR}/${_script}
    COPYONLY
    )
endforeach()

#----------------------------------------------------------------------------
# Install the executable to 'bin' directory under CMAKE_INSTALL_PREFIX
#
install(TARGETS padmpi RUNTIME DESTINATION bin)
