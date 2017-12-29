cmake_minimum_required(VERSION 2.8.12)
project(brutefir)
find_package(FLEX)
find_package(ALSA)
find_library(FFTW_LIBRARY   NAMES fftw3)
find_library(FFTW_LIBRARY_F NAMES fftw3f)


set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
# define a default build type
if (NOT CMAKE_BUILD_TYPE)
  message(STATUS "No build type selected, default to Release")
  set(CMAKE_BUILD_TYPE "Release")
endif()

find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
  message(STATUS "CCACHE enabled")
endif(CCACHE_FOUND)

find_program(MEMORYCHECK_COMMAND NAMES valgrind)
set( MEMORYCHECK_COMMAND_OPTIONS "--max-stackframe=9000000 -v" )

flex_target(LEXER bfconf_lexical.lex ${CMAKE_CURRENT_BINARY_DIR}/bfconf_lexical.c)

add_executable(brutefir
  brutefir.c
  fftw_convolver.c
  bfconf.c
  bfrun.c
  firwindow.c
  emalloc.c
  shmalloc.c
  dai.c
  bfconf_lexical.c
  inout.c
  dither.c
  delay.c
  convolver_xmm.c
  ${FLEX_LEXER_OUTPUTS}
  )

if(ALSA_FOUND)
  add_library(alsa.bfio SHARED
    bfio_alsa.c emalloc.c inout.c
    )

  target_compile_options(alsa.bfio PRIVATE -fPIC)
  target_link_libraries(alsa.bfio PRIVATE ${ALSA_LIBRARY})
  set_target_properties(alsa.bfio PROPERTIES PREFIX "")
  set_target_properties(alsa.bfio PROPERTIES SUFFIX "")

  INSTALL(TARGETS brutefir alsa.bfio
	ARCHIVE DESTINATION /usr/lib
	LIBRARY DESTINATION /usr/lib
	RUNTIME DESTINATION bin
)

endif(ALSA_FOUND)


add_library(file.bfio SHARED
  bfio_file.c emalloc.c inout.c
  )

target_compile_options(file.bfio PRIVATE -fPIC)
#target_link_libraries(file.bfio PRIVATE ${FILE_LIBRARY})
set_target_properties(file.bfio PROPERTIES PREFIX "")
set_target_properties(file.bfio PROPERTIES SUFFIX "")
#set_target_properties(file.bfio PROPERTIES OUTPUT_NAME file.bfio)


add_library(eq.bflogic SHARED
  bflogic_eq.c emalloc.c shmalloc.c
  )

target_compile_options(eq.bflogic PRIVATE -fPIC)
set_target_properties(eq.bflogic PROPERTIES PREFIX "")
set_target_properties(eq.bflogic PROPERTIES SUFFIX "")

add_library(cli.bflogic SHARED
  bflogic_cli.c inout.c
  )

target_compile_options(cli.bflogic PRIVATE -fPIC)
set_target_properties(cli.bflogic PROPERTIES PREFIX "")
set_target_properties(cli.bflogic PROPERTIES SUFFIX "")



target_include_directories(brutefir PRIVATE
  ${CMAKE_CURRENT_BINARY_DIR}
  .
  )
target_link_libraries(brutefir ${FFTW_LIBRARY} ${FFTW_LIBRARY_F} dl m)

INSTALL(TARGETS brutefir file.bfio eq.bflogic cli.bflogic
	ARCHIVE DESTINATION /usr/lib
	LIBRARY DESTINATION /usr/lib
	RUNTIME DESTINATION bin
)
