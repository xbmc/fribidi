cmake_minimum_required(VERSION 3.2)

project(FriBidi C)

set(FRIBIDI_MAJOR_VERSION 0)
set(FRIBIDI_MINOR_VERSION 19)
set(FRIBIDI_MICRO_VERSION 7)
set(FRIBIDI_INTERFACE_VERSION 3)
set(FRIBIDY_INTERFACE_AGE 6)
set(FRIBIDI_BINARY_AGE 3)

set(FRIBIDI_VERSION "${FRIBIDI_MAJOR_VERSION}.${FRIBIDI_MINOR_VERSION}.${FRIBIDI_MICRO_VERSION}")
set(FRIBIDI_CHARSETS 0)
set(FRIBIDI_USE_GLIB 0)
set(FRIBIDI_SIZEOF_INT 4)


configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-config.h.in
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-config.h
  @ONLY
)

set(COMPRESSION 2)

add_executable(gen-unicode-version
  gen.tab/gen-unicode-version.c)
target_include_directories(gen-unicode-version PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib>
)
add_custom_target(gen-unicode
  COMMAND gen-unicode-version
  ${CMAKE_CURRENT_SOURCE_DIR}/gen.tab/unidata/ReadMe.txt >
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-unicode-version.h
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib
  BYPRODUCTS ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-unicode-version.h
  )

add_executable(gen-bidi-type-tab
  gen.tab/gen-bidi-type-tab.c
  gen.tab/packtab.h
  gen.tab/packtab.c)
target_include_directories(gen-bidi-type-tab PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib>
)
add_dependencies(gen-bidi-type-tab gen-unicode)

add_custom_target(gen-derived-bidi-type
  COMMAND gen-bidi-type-tab
  ${COMPRESSION}
  ${CMAKE_CURRENT_SOURCE_DIR}/gen.tab/unidata/DerivedBidiClass.txt >
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/derived_bidi-type.tab.i
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib
  BYPRODUCTS ${CMAKE_CURRENT_SOURCE_DIR}/lib/derived_bidi-type.tab.i)

add_custom_target(gen-bidi-type
  COMMAND gen-bidi-type-tab
  ${COMPRESSION}
  ${CMAKE_CURRENT_SOURCE_DIR}/gen.tab/unidata/UnicodeData.txt >
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/bidi-type.tab.i
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib
  BYPRODUCTS ${CMAKE_CURRENT_SOURCE_DIR}/lib/bidi-type.tab.i)

add_executable(gen-joining-type-tab
  gen.tab/packtab.h
  gen.tab/packtab.c
  gen.tab/gen-joining-type-tab.c)
target_include_directories(gen-joining-type-tab PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib>
)
add_dependencies(gen-joining-type-tab gen-unicode)

add_custom_target(gen-joining-type
  COMMAND gen-joining-type-tab
  ${COMPRESSION}
  ${CMAKE_CURRENT_SOURCE_DIR}/gen.tab/unidata/UnicodeData.txt
  ${CMAKE_CURRENT_SOURCE_DIR}/gen.tab/unidata/ArabicShaping.txt >
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/joining-type.tab.i
  BYPRODUCTS ${CMAKE_CURRENT_SOURCE_DIR}/lib/joining-type.tab.i)

add_executable(gen-arabic-shaping-tab
  gen.tab/gen-arabic-shaping-tab.c)
target_include_directories(gen-arabic-shaping-tab PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib>
)
add_dependencies(gen-arabic-shaping-tab gen-unicode)

add_custom_target(gen-arabic-shaping
  COMMAND gen-arabic-shaping-tab
  ${COMPRESSION}
  ${CMAKE_CURRENT_SOURCE_DIR}/gen.tab/unidata/UnicodeData.txt >
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/arabic-shaping.tab.i
  BYPRODUCTS ${CMAKE_CURRENT_SOURCE_DIR}/lib/arabic-shaping.tab.i)

add_executable(gen-mirroring-tab
  gen.tab/packtab.h
  gen.tab/packtab.c
  gen.tab/gen-mirroring-tab.c)
target_include_directories(gen-mirroring-tab PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib>
)
add_dependencies(gen-mirroring-tab gen-unicode)

add_custom_target(gen-mirroring
  COMMAND gen-mirroring-tab
  ${COMPRESSION}
  ${CMAKE_CURRENT_SOURCE_DIR}/gen.tab/unidata/BidiMirroring.txt >
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/mirroring.tab.i
  BYPRODUCTS ${CMAKE_CURRENT_SOURCE_DIR}/lib/mirroring.tab.i)

add_library(fribidi STATIC
  lib/bidi-types.h
  lib/common.h
  lib/debug.h
  lib/fribidi.c
  lib/fribidi.def
  lib/fribidi.h
  lib/fribidi-arabic.h
  lib/fribidi-arabic.c
  lib/fribidi-begindecls.h
  lib/fribidi-bidi.h
  lib/fribidi-bidi.c
  lib/fribidi-bidi-types.h
  lib/fribidi-bidi-types.c
  lib/fribidi-bidi-types-list.h
  lib/fribidi-common.h
  lib/fribidi-config.h
  lib/fribidi-deprecated.h
  lib/fribidi-deprecated.c
  lib/fribidi-enddecls.h
  lib/fribidi-joining.h
  lib/fribidi-joining.c
  lib/fribidi-joining-types.h
  lib/fribidi-joining-types.c
  lib/fribidi-joining-types-list.h
  lib/fribidi-mirroring.h
  lib/fribidi-mirroring.c
  lib/fribidi-run.c
  lib/fribidi-shape.h
  lib/fribidi-shape.c
  lib/fribidi-types.h
  lib/fribidi-unicode.h
  lib/joining-types.h
  lib/run.h
  )
target_include_directories(fribidi
  PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib>
  INTERFACE
  $<INSTALL_INTERFACE:include/fribidi>
)
add_dependencies(fribidi
  gen-unicode
  gen-mirroring
  gen-arabic-shaping
  gen-joining-type
  # gen-derived-bidi-type
  gen-bidi-type
  )

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/fribidi-config-version.cmake
  VERSION ${FRIBIDI_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(TARGETS fribidi EXPORT fribidi
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib)


install(FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-arabic.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-begindecls.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-bidi.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-bidi-types.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-bidi-types-list.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-common.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-config.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-deprecated.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-enddecls.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-flags.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-joining.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-joining-types.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-joining-types-list.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-mirroring.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-shape.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-types.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-unicode.h
  ${CMAKE_CURRENT_SOURCE_DIR}/lib/fribidi-unicode-version.h
  DESTINATION include/fribidi)

install(EXPORT fribidi
  FILE
    fribidi.cmake
  NAMESPACE
    fribidi::
  DESTINATION
    lib/cmake/fribidi
)
install(
  FILES
    cmake/fribidi-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/fribidi-config-version.cmake
  DESTINATION
    lib/cmake/fribidi
)