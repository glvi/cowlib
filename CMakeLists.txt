# -*- mode: cmake; coding: utf-8; indent-tabs-mode: nil; -*-
project(cowlib
  VERSION 1.0.0
  DESCRIPTION "Cowlib web socket utilities"
  LANGUAGES NONE)
set(cowlib_source_dir
  ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(COWLIB_MODULES
  cow_base64url
  cow_cookie
  cow_date
  cow_hpack
  cow_http
  cow_http2
  cow_http_hd
  cow_http_te
  cow_iolists
  cow_mimetypes
  cow_multipart
  cow_qs
  cow_spdy
  cow_sse
  cow_uri
  cow_ws)
foreach(module IN LISTS COWLIB_MODULES)
  list(APPEND COWLIB_SOURCES    ${cowlib_source_dir}/${module}.erl)
  list(APPEND COWLIB_OBJECTS    ${CMAKE_CURRENT_BINARY_DIR}/ebin/${module}.beam)
  list(APPEND COWLIB_MODULES_Q '${module}')
endforeach()
file(RELATIVE_PATH cowlib_include_dir ${cowlib_source_dir} ${CMAKE_CURRENT_SOURCE_DIR}/include)
file(RELATIVE_PATH cowlib_binary_dir  ${cowlib_source_dir} ${CMAKE_CURRENT_BINARY_DIR}/ebin)
string(REPLACE ";" "," cowlib_modules "${COWLIB_MODULES_Q}")
configure_file(Emakefile.in ${cowlib_source_dir}/Emakefile)
configure_file(${cowlib_source_dir}/cowlib.app.src ebin/cowlib.app)
add_custom_target(CowLib
  BYPRODUCTS ${COWLIB_OBJECTS}
  DEPENDS    ${COWLIB_SOURCES}
  COMMAND    ${CMAKE_COMMAND} -E make_directory ${cowlib_binary_dir}
  COMMAND    ${CMAKE_COMMAND} -E copy_directory ${cowlib_include_dir} ${CMAKE_CURRENT_BINARY_DIR}
  COMMAND    ${ERLANG_PROGRAM} -sasl errlog_type error -make
  WORKING_DIRECTORY ${cowlib_source_dir}
  VERBATIM
  COMMAND_EXPAND_LISTS)
#add_dependencies(CowLib ErlangOTP)
set(URBANATM_COWLIB_OBJECTS ${COWLIB_OBJECTS} CACHE INTERNAL "UrbanATM Erlang CowLib objects" FORCE)
add_custom_target(DialyzeCowLib
  COMMAND ${DIALYZER_PROGRAM}
          --fullpath
          --plt ${DIALYZER_PLT}
          -I ${CMAKE_CURRENT_SOURCE_DIR}/include
          -pa ${CMAKE_CURRENT_BINARY_DIR}/ebin
          --src ${CMAKE_CURRENT_SOURCE_DIR}/src
  DEPENDS ${DIALYZER_PLT}
  )
add_dependencies(DialyzeCowLib
  DialyzerPlt
  CowLib)
