IF(CONNECTOR_HTTP)
  MESSAGE("** Enabling built-in httpd.")

  SET(libhttpsources
    Android.C
    Configuration.C
    Connection.C
    ConnectionManager.C
    HTTPRequest.C
    MimeTypes.C
    Reply.C
    Request.C
    RequestHandler.C
    RequestParser.C
    Server.C
    SslConnection.C
    StaticReply.C
    StockReply.C
    TcpConnection.C
    WServer.C
    WtReply.C
  )

 OPTION(HTTP_WITH_ZLIB "Support for zlib (http compression)" ${ZLIB_FOUND})

 IF (HAVE_SSL)
    SET(MY_SSL_LIBS ${SSL_LIBRARIES})
    ADD_DEFINITIONS(-DHTTP_WITH_SSL)
 ELSE (HAVE_SSL)
    SET(MY_SSL_LIBS "")
 ENDIF (HAVE_SSL)

 IF(NOT BOOST_WTHTTP_FOUND)

    MESSAGE(FATAL "** Could not find all boost libraries required to build the httpd connector (thread, filesystem, programoptions, datetime)")

  ENDIF(NOT BOOST_WTHTTP_FOUND)
  IF(HTTP_WITH_ZLIB)
    ADD_DEFINITIONS(-DWTHTTP_WITH_ZLIB ${ZLIB_DEFINITIONS})
    SET(MY_ZLIB_LIBS ${ZLIB_LIBRARIES})
    INCLUDE_DIRECTORIES(${ZLIB_INCLUDE_DIRS})
  ELSE(HTTP_WITH_ZLIB)
    SET(MY_ZLIB_LIBS "")
  ENDIF(HTTP_WITH_ZLIB)

  INCLUDE_DIRECTORIES(
    ${BOOST_INCLUDE_DIRS}
    ${CMAKE_CURRENT_SOURCE_DIR}/../web
    ${CMAKE_CURRENT_SOURCE_DIR}/../wt
    ${CMAKE_CURRENT_BINARY_DIR}/
  )

  ADD_DEFINITIONS(${ZLIB_DEFINITIONS})
  ADD_LIBRARY(wthttp ${libhttpsources})

  TARGET_LINK_LIBRARIES(wthttp
    wt
    ${MY_ZLIB_LIBS}
    ${MY_SSL_LIBS}
    ${BOOST_WTHTTP_LIBRARIES}
    ${WT_SOCKET_LIBRARY}
  )

  IF(BOOST_WT_MT_FOUND)
    TARGET_LINK_LIBRARIES(wthttp ${CMAKE_THREAD_LIBS_INIT})
  ENDIF(BOOST_WT_MT_FOUND)

  INSTALL(TARGETS wthttp
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION ${LIB_INSTALL_DIR}
    ARCHIVE DESTINATION ${LIB_INSTALL_DIR})

  SET_TARGET_PROPERTIES(
    wthttp
  PROPERTIES
    VERSION ${VERSION_SERIES}.${VERSION_MAJOR}.${VERSION_MINOR}
    SOVERSION ${WTHTTP_SOVERSION}
    DEBUG_POSTFIX ${DEBUG_LIB_POSTFIX}
  )

  IF(MSVC)
    SET_TARGET_PROPERTIES(
        wthttp
      PROPERTIES
        COMPILE_FLAGS "${BUILD_PARALLEL} /wd4251 /wd4275 /wd4355 /wd4800 /wd4996 /wd4101 /wd4267 /wd4267"
    )
  ENDIF(MSVC)

  ADD_DEPENDENCIES(wthttp wt ${EXAMPLES_CONNECTOR})

ELSE(CONNECTOR_HTTP)
  MESSAGE("** Disabling built-in httpd.")
ENDIF(CONNECTOR_HTTP)

