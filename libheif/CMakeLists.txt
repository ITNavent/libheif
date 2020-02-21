include(CMakePackageConfigHelpers)

configure_file(heif_version.h.in ${CMAKE_CURRENT_BINARY_DIR}/heif_version.h)

find_package(JNI REQUIRED)

set(CMAKE_CXX_FLAGS "-fPIC -I$ENV{JAVA_HOME}/include -I$ENV{JAVA_HOME}/include/linux")

set(libheif_headers
    bitstream.h
    box.h
    error.h
    heif_api_structs.h
    heif_context.h
    heif_file.h
    heif.h
    heif_image.h
    heif_hevc.h
    heif_colorconversion.h
    heif_plugin_registry.h
    heif_limits.h
    heif_plugin.h
    logging.h
    encoder_jpeg.h
    encoder_png.h
    encoder_y4m.h
    encoder.h
    com_navent_realestate_converter_HEIFConverter.h
    ${CMAKE_CURRENT_BINARY_DIR}/heif_version.h
)

add_library(heif
    bitstream.cc
    box.cc
    error.cc
    heif.cc
    heif_context.cc
    heif_file.cc
    heif_image.cc
    heif_hevc.cc
    heif_colorconversion.cc
    heif_plugin_registry.cc
    heif_plugin.cc
    encoder_jpeg.cc
    encoder_png.cc
    encoder_y4m.cc
    encoder.cc
    HEIFConverter.cc
    ${libheif_headers}
)

# Needed to find libheif/heif_version.h while compiling the library
target_include_directories(heif PRIVATE ${libheif_BINARY_DIR} ${libheif_SOURCE_DIR})

# Propagate include/libheif to consume the headers from other projects
target_include_directories(heif
                           PUBLIC
                               $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include/${PROJECT_NAME}>
                               $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
                               $<INSTALL_INTERFACE:include>)

set_target_properties(heif
                      PROPERTIES
                          VERSION ${PROJECT_VERSION}
                          SOVERSION ${PROJECT_VERSION_MAJOR})

target_compile_definitions(heif
                           PUBLIC
                               LIBHEIF_EXPORTS
                               HAVE_VISIBILITY)

if(LIBDE265_FOUND)
    target_compile_definitions(heif PRIVATE HAVE_LIBDE265=1)
    target_sources(heif
                   PRIVATE
                       heif_decoder_libde265.cc
                       heif_decoder_libde265.h)

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${LIBDE265_CFLAGS}")

    if (NOT "${LIBDE265_LIBRARY_DIRS}" STREQUAL "")
        set(LIBDE265_LINKDIR "-L${LIBDE265_LIBRARY_DIRS}")
    endif()

    target_link_libraries(heif PRIVATE ${LIBDE265_LIBRARIES} ${LIBDE265_LINKDIR})

    install(FILES heif_decoder_libde265.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})
endif()

if(X265_FOUND)
    target_compile_definitions(heif PRIVATE HAVE_X265=1)
    target_sources(heif PRIVATE
        heif_encoder_x265.cc
        heif_encoder_x265.h
    )
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${X265_CFLAGS}")

    if (NOT "${X265_LIBRARY_DIRS}" STREQUAL "")
        set(X265_LINKDIR "-L${X265_LIBRARY_DIRS}")
    endif()

    target_link_libraries(heif PRIVATE ${X265_LIBRARIES} ${X265_LINKDIR})

    install(FILES heif_encoder_x265.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})
endif()

set (additional_libraries)

include (${CMAKE_ROOT}/Modules/FindJPEG.cmake)

if(JPEG_FOUND)
add_definitions(-DHAVE_LIBJPEG=1)

include (${CMAKE_ROOT}/Modules/CheckCXXSourceCompiles.cmake)

set(CMAKE_REQUIRED_LIBRARIES ${JPEG_LIBRARIES})
check_cxx_source_compiles("
#include <stddef.h>
#include <stdio.h>
#include <jpeglib.h>

int main() {
  jpeg_write_icc_profile(NULL, NULL, 0);
  return 0;
}
" HAVE_JPEG_WRITE_ICC_PROFILE)
if(HAVE_JPEG_WRITE_ICC_PROFILE)
  add_definitions(-DHAVE_JPEG_WRITE_ICC_PROFILE=1)
endif()

set (heif_convert_sources
  ${heif_convert_sources}
  encoder_jpeg.cc
  encoder_jpeg.h
)
target_link_libraries(heif PRIVATE ${JPEG_LIBRARIES})
endif()

if(UNIX)
  include (${CMAKE_ROOT}/Modules/FindPkgConfig.cmake)
  pkg_check_modules (LIBPNG libpng)
  if(LIBPNG_FOUND)
    add_definitions(-DHAVE_LIBPNG=1)
    set (heif_convert_sources
      ${heif_convert_sources}
      encoder_png.cc
      encoder_png.h
    )

    target_link_libraries(heif PRIVATE ${LIBPNG_LIBRARIES})
  endif()
endif()

if(MSVC)
  set (getopt_sources
    ../extra/getopt.c
    ../extra/getopt.h
    ../extra/getopt_long.c
  )
  include_directories ("../extra")
endif()

#target_link_libraries(heif ${additional_libraries})

write_basic_package_version_file(${PROJECT_NAME}-config-version.cmake COMPATIBILITY ExactVersion)

install(TARGETS heif EXPORT ${PROJECT_NAME}-config
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(FILES ${libheif_headers} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})

install(EXPORT ${PROJECT_NAME}-config DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake DESTINATION
    "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")
