cmake_minimum_required(VERSION 3.1.0 FATAL_ERROR)

project(eiger2cbf LANGUAGES C)

# Set the standard level of C
set(CMAKE_C_STANDARD 99)

# Use 'Release with symbols' configuration unless specifically requested
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  # (see https://blog.kitware.com/cmake-and-the-default-build-type/ )
  set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

########################################################################
# Find Zlib
find_package(ZLIB REQUIRED)

########################################################################
# Find HDF5 - preferably static, and create an imported target
option(HDF5_USE_STATIC_LIBRARIES YES)
set(HDF5_FIND_DEBUG TRUE)
find_package(HDF5 COMPONENTS HL REQUIRED)
# Need to create our own imported library as FindHDF5 doesn't provide
string(REGEX REPLACE "(^|\\s|;)-D" ";" HDF5_DEFINITIONS_CLEAN    "${HDF5_DEFINITIONS}")
string(REGEX REPLACE "(^|\\s|;)-D" ";" HDF5_C_DEFINITIONS_CLEAN  "${HDF5_C_DEFINITIONS}")
string(REGEX REPLACE "(^|\\s|;)-D" ";" HDF5_HL_DEFINITIONS_CLEAN  "${HDF5_HL_DEFINITIONS}")

add_library(HDF5::HDF5 INTERFACE IMPORTED)
set_target_properties(HDF5::HDF5 PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES "${HDF5_INCLUDE_DIRS}"
  INTERFACE_COMPILE_DEFINITIONS "${HDF5_DEFINITIONS_CLEAN}"
  INTERFACE_LINK_LIBRARIES      "${HDF5_LIBRARIES}" )
# set_target_properties(HDF5::CXX PROPERTIES
#   INTERFACE_INCLUDE_DIRECTORIES "${HDF5_CXX_INCLUDE_DIRS}"
#   INTERFACE_COMPILE_DEFINITIONS "${HDF5_CXX_DEFINITIONS}"
#   INTERFACE_LINK_LIBRARIES      "${HDF5_CXX_LIBRARIES};HDF5::HDF5" )
# add_dependencies(HDF5::CXX HDF5::HDF5)
add_library(HDF5::C INTERFACE IMPORTED)
set_target_properties(HDF5::C PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES "${HDF5_C_INCLUDE_DIRS}"
  INTERFACE_COMPILE_DEFINITIONS "${HDF5_C_DEFINITIONS_CLEAN}"
  INTERFACE_LINK_LIBRARIES      "${HDF5_C_LIBRARIES};HDF5::HDF5")
add_dependencies(HDF5::C HDF5::HDF5)

add_library(HDF5::HL INTERFACE IMPORTED)
set_target_properties(HDF5::HL PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES "${HDF5_HL_INCLUDE_DIRS}"
  INTERFACE_COMPILE_DEFINITIONS "${HDF5_HL_DEFINITIONS_CLEAN}"
  INTERFACE_LINK_LIBRARIES      "${HDF5_HL_LIBRARIES};HDF5::HDF5")
add_dependencies(HDF5::HL HDF5::HDF5)

# message("HDF5:")
# message("      i: ${HDF5_INCLUDE_DIRS}")
# message("      D: ${HDF5_DEFINITIONS_CLEAN}")
# message("      l: ${HDF5_LIBRARIES}")
# message(" CXX: i: ${HDF5_CXX_INCLUDE_DIRS}")
# message("      D: ${HDF5_CXX_DEFINITIONS}")
# message("      l: ${HDF5_CXX_LIBRARIES}")
# message("   C: i: ${HDF5_C_INCLUDE_DIRS}")
# message("      D: ${HDF5_C_DEFINITIONS_CLEAN}")
# message("      l: ${HDF5_C_LIBRARIES}")
# message("  HL: i: ${HDF5_HL_INCLUDE_DIRS}")
# message("      D: ${HDF5_HL_DEFINITIONS_CLEAN}")
# message("      l: ${HDF5_HL_LIBRARIES}")

########################################################################
# message(WARNING "CBFlib not configured yet; compile will fail")
message(WARNING "Using hardcoded cbflib location")
# Create a manual cbflib for now
add_library(CBF::CBF INTERFACE IMPORTED)
set_target_properties(CBF::CBF PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES "/dls/science/users/mep23677/dials_pure/modules/cbflib/include"
  INTERFACE_LINK_LIBRARIES      "/dls/science/users/mep23677/dials_pure/build/lib/libcbf.a")

########################################################################
# Build the main program
add_executable(eiger2cbf  eiger2cbf.c 
                          lz4/lz4.c 
                          lz4/h5zlz4.c
                          bitshuffle/bshuf_h5filter.c
                          bitshuffle/bshuf_h5plugin.c
                          bitshuffle/bitshuffle.c )
target_include_directories(eiger2cbf PRIVATE lz4 bitshuffle)
target_link_libraries(eiger2cbf PRIVATE ZLIB::ZLIB HDF5::HL CBF::CBF)
