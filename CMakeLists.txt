cmake_minimum_required(VERSION 3.22.1)

# Uncomment for (gdbserver) debugging
if(BOARD MATCHES "qemu_.*")
  list(APPEND QEMU_EXTRA_FLAGS -serial tcp:127.0.0.1:5678,server)
endif()

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(zenoh_hello_world)

include(ExternalProject)
include(CMakePrintHelpers)

zephyr_get_system_include_directories_for_lang_as_string(C ext_system_includes)
zephyr_get_include_directories_for_lang_as_string(C ext_includes)
zephyr_get_compile_definitions_for_lang_as_string(C ext_defs)
zephyr_get_compile_options_for_lang_as_String(C ext_opts)
set(ZENOHPICO_CFLAGS
	"${ext_defs} ${ext_system_includes} ${ext_includes} ${ext_opts}"
)

set(ZENOHPICO_LIB_DIR ${CMAKE_CURRENT_BINARY_DIR}/zenoh-pico-prefix/lib)
set(ZENOHPICO_INCLUDE_DIR ${CMAKE_CURRENT_BINARY_DIR}/zenoh-pico-prefix/include)

ExternalProject_Add(zenoh-pico
  SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/zenoh-pico
  BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/zenoh-pico
  BUILD_COMMAND ${CMAKE_COMMAND} --build .
  CMAKE_ARGS
  -DWITH_ZEPHYR=1 -DBUILD_INTEGRATION=OFF -DBUILD_TOOLS=OFF -DBUILD_TESTING=OFF -DBUILD_MULTICAST=OFF
    -DCMAKE_TRY_COMPILE_TARGET_TYPE=STATIC_LIBRARY
    -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
    -DCMAKE_VERBOSE_MAKEFILE=1
    -DCMAKE_C_FLAGS=${ZENOHPICO_CFLAGS}
    -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
    -DCMAKE_SYSTEM_NAME=Generic
    -DCMAKE_BUILD_TYPE=Debug
  DEPENDS zephyr_interface
  BUILD_BYPRODUCTS ${ZENOHPICO_LIB_DIR}/libzenohpico.a
)
add_library(zenohpico_lib STATIC IMPORTED GLOBAL)
add_dependencies(zenohpico_lib zenoh-pico)
set_target_properties(zenohpico_lib PROPERTIES IMPORTED_LOCATION ${ZENOHPICO_LIB_DIR}/libzenohpico.a)
add_definitions(-DZENOH_ZEPHYR)

target_include_directories(app PUBLIC ${ZENOHPICO_INCLUDE_DIR})
target_link_libraries(app PUBLIC zenohpico_lib)

target_sources(app
    PRIVATE
    src/main.c
)
