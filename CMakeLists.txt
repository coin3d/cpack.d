# get all found components
get_cmake_property(PACKAGE_COMPONENTS_ALL COMPONENTS)

find_program(HG_LOCATION NAMES hg)
if (HG_LOCATION)
  # calculates the build number as the number of commits on the current branch
  execute_process(
    COMMAND ${HG_LOCATION} id --num --rev tip
    WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}"
    OUTPUT_VARIABLE PACKAGE_BUILD_NUMBER
  )
  string(REGEX REPLACE "\n$" "" PACKAGE_BUILD_NUMBER "${PACKAGE_BUILD_NUMBER}")
  message(STATUS "Mercurial found at ${HG_LOCATION}, PACKAGE_BUILD_NUMBER set to ${PACKAGE_BUILD_NUMBER}")
else()
  set(PACKAGE_BUILD_NUMBER 1)
  message(WARNING "Mercurial not found, PACKAGE_BUILD_NUMBER set to ${PACKAGE_BUILD_NUMBER}")
endif()
  
configure_file(commons.cmake.in commons.cmake @ONLY)
configure_file(fedora.cmake.in fedora.cmake @ONLY)
configure_file(debian.cmake.in debian.cmake @ONLY)
