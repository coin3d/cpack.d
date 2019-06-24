# retrieves mercurial project infos setting the following variables:
#
# PACKAGE_BRANCH  current hg branch name
# PACKAGE_COMMITS number of commits on the current branch
function(hg_info)
  execute_process(
    COMMAND ${HG_LOCATION} branch
    WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}"
    OUTPUT_VARIABLE BRANCH
  )
  string(REGEX REPLACE "\n$" "" BRANCH "${BRANCH}")
  set(PACKAGE_BRANCH "${BRANCH}" PARENT_SCOPE)
  execute_process(
    COMMAND ${HG_LOCATION} log -b ${BRANCH} --template .
    WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}"
    OUTPUT_VARIABLE DUMMY
  )
  string(LENGTH "${DUMMY}" PACKAGE_BUILD_NUMBER)
  set(PACKAGE_BUILD_NUMBER "${PACKAGE_BUILD_NUMBER}" PARENT_SCOPE)
endfunction(hg_info)

# get all found components
get_cmake_property(PACKAGE_COMPONENTS_ALL COMPONENTS)

find_program(HG_LOCATION NAMES hg)
if (HG_LOCATION)
  hg_info()
  message(STATUS "Mercurial found at ${HG_LOCATION}, PACKAGE_BUILD_NUMBER set to ${PACKAGE_BUILD_NUMBER}")
else()
  set(PACKAGE_BUILD_NUMBER 1)
  message(WARNING "Mercurial not found, PACKAGE_BUILD_NUMBER set to ${PACKAGE_BUILD_NUMBER}")
endif()
  
file(GLOB CONFIGS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.cmake.in)

if(APPLE)
  configure_file(DMGSetup.scpt.in DMGSetup.scpt @ONLY)
  configure_file("${CMAKE_SOURCE_DIR}/COPYING" "${CMAKE_BINARY_DIR}/COPYING.txt" COPYONLY)
  configure_file("${CMAKE_SOURCE_DIR}/README"  "${CMAKE_BINARY_DIR}/README.txt"  COPYONLY)
  configure_file("${CMAKE_SOURCE_DIR}/packaging/macosx/Welcome.rtf"  "${CMAKE_BINARY_DIR}/WELCOME.rtf"  COPYONLY)
endif()

foreach(FILE ${CONFIGS})
  string(REGEX REPLACE ".cmake.in$" ".cmake" CFG ${FILE})
  configure_file(${FILE} ${CFG} @ONLY)
endforeach()
