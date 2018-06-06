file(GLOB CONFIGS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.cmake.in)

foreach(FILE ${CONFIGS})
  string(REGEX REPLACE ".cmake.in$" ".cmake" CFG ${FILE})
  configure_file(${FILE} ${CFG} @ONLY)
endforeach()

