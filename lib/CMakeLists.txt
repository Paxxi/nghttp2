add_subdirectory(includes)

include_directories(
  "${CMAKE_CURRENT_SOURCE_DIR}/includes"
  "${CMAKE_CURRENT_BINARY_DIR}/includes"
)

add_definitions(-DBUILDING_NGHTTP2)

set(NGHTTP2_SOURCES
  nghttp2_pq.c nghttp2_map.c nghttp2_queue.c
  nghttp2_frame.c
  nghttp2_buf.c
  nghttp2_stream.c nghttp2_outbound_item.c
  nghttp2_session.c nghttp2_submit.c
  nghttp2_helper.c
  nghttp2_npn.c
  nghttp2_hd.c nghttp2_hd_huffman.c nghttp2_hd_huffman_data.c
  nghttp2_version.c
  nghttp2_priority_spec.c
  nghttp2_option.c
  nghttp2_callbacks.c
  nghttp2_mem.c
  nghttp2_http.c
  nghttp2_rcbuf.c
  nghttp2_debug.c
)

set(NGHTTP2_RES "")

if(WIN32)
  configure_file(
    version.rc.in
    ${CMAKE_CURRENT_BINARY_DIR}/version.rc
    @ONLY)

  set(NGHTTP2_RES ${CMAKE_CURRENT_BINARY_DIR}/version.rc)
endif()

# Public shared library
if(ENABLE_SHARED_LIB)
  add_library(nghttp2 SHARED ${NGHTTP2_SOURCES} ${NGHTTP2_RES})
  set_target_properties(nghttp2 PROPERTIES
    COMPILE_FLAGS "${WARNCFLAGS}"
    VERSION ${LT_VERSION} SOVERSION ${LT_SOVERSION}
    C_VISIBILITY_PRESET hidden
    COMPILE_PDB_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}
    PDB_NAME nghttp2
  )
  target_include_directories(nghttp2 INTERFACE
    "${CMAKE_CURRENT_BINARY_DIR}/includes"
    "${CMAKE_CURRENT_SOURCE_DIR}/includes"
  )

  install(TARGETS nghttp2 EXPORT nghttp2
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")
  install(FILES
    $<TARGET_PDB_FILE:nghttp2>
    DESTINATION lib
  )
    install(EXPORT nghttp2
      NAMESPACE nghttp2::
      DESTINATION lib/cmake/nghttp2
    )
endif()

if(HAVE_CUNIT OR ENABLE_STATIC_LIB)
  # Static library (for unittests because of symbol visibility)
  add_library(nghttp2_static STATIC ${NGHTTP2_SOURCES})
  set_target_properties(nghttp2_static PROPERTIES
    COMPILE_FLAGS "${WARNCFLAGS}"
    VERSION ${LT_VERSION} SOVERSION ${LT_SOVERSION}
    ARCHIVE_OUTPUT_NAME nghttp2_static
    ARCHIVE_OUTPUT_NAME_DEBUG nghttp2_staticd
    COMPILE_PDB_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}
    COMPILE_PDB_NAME nghttp2_static
    COMPILE_PDB_NAME_DEBUG nghttp2_staticd
  )
  target_compile_definitions(nghttp2_static PUBLIC "-DNGHTTP2_STATICLIB")
  if(ENABLE_STATIC_LIB)
    install(TARGETS nghttp2_static EXPORT nghttp2_static
      DESTINATION "${CMAKE_INSTALL_LIBDIR}")
    install(FILES
      ${PROJECT_BINARY_DIR}/RelWithDebInfo/nghttp2_static.pdb
      DESTINATION lib
      CONFIGURATIONS RelWithDebInfo
    )
    install(FILES
      ${PROJECT_BINARY_DIR}/Debug/nghttp2_staticd.pdb
      DESTINATION lib
      CONFIGURATIONS Debug
    )
    install(EXPORT nghttp2_static
      NAMESPACE nghttp2::
      DESTINATION lib/cmake/nghttp2
    )
  endif()
endif()


install(FILES "${CMAKE_CURRENT_BINARY_DIR}/libnghttp2.pc"
  DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(FILES
  ${CMAKE_SOURCE_DIR}/cmake/${PROJECT_NAME}-config.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  DESTINATION
    lib/cmake/${PROJECT_NAME}
)