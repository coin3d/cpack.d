# retrieves mercurial project infos setting the following variables:
#
# PACKAGE_BRANCH  current hg branch name
# PACKAGE_COMMITS number of commits on the current branch
function(hg_info)
  # hg command from mercurial package under msys2 is a python2 script.
  # As execute_process on MSYS2/MinGW uses CMD.exe as command interpreter
  # this cannot be started.
  set(HG_COMMAND ${HG_EXECUTABLE})
  if (MSYS OR MINGW)
    execute_process(
      COMMAND file ${HG_EXECUTABLE}
      WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
      OUTPUT_STRIP_TRAILING_WHITESPACE
      OUTPUT_VARIABLE hg_file_type
    )
    if ("${hg_file_type}" MATCHES "Python script")
      set(HG_COMMAND python2 ${HG_EXECUTABLE})
    endif()
  endif()
  execute_process(
    COMMAND ${HG_COMMAND} branch
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
    OUTPUT_STRIP_TRAILING_WHITESPACE
    OUTPUT_VARIABLE BRANCH
  )
  set(PACKAGE_BRANCH "${BRANCH}" PARENT_SCOPE)
  execute_process(
    COMMAND ${HG_COMMAND} log -b ${BRANCH} --template .
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
    OUTPUT_STRIP_TRAILING_WHITESPACE
    OUTPUT_VARIABLE DUMMY
  )
  string(LENGTH "${DUMMY}" PACKAGE_BUILD_NUMBER)
  set(PACKAGE_BUILD_NUMBER "${PACKAGE_BUILD_NUMBER}" PARENT_SCOPE)
endfunction(hg_info)

# get all found components
get_cmake_property(PACKAGE_COMPONENTS_ALL COMPONENTS)

find_program(HG_EXECUTABLE NAMES hg)
if (HG_EXECUTABLE)
  hg_info()
  message(STATUS "Mercurial found at ${HG_EXECUTABLE}, PACKAGE_BUILD_NUMBER set to ${PACKAGE_BUILD_NUMBER}")
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
