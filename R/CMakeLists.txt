#oli: I changed this to FindR which seems quite stable here
find_package(R REQUIRED)

OPTION(BUILD_R_PACKAGE "Build the peer R package" ON)

if (BUILD_R_PACKAGE)
  find_package(SWIG REQUIRED)

  INCLUDE(${SWIG_USE_FILE})

  include_directories(./../External)
  include_directories(./../External/Eigen)
  include_directories(${PEER_SRC_DIR}/External/alglib)
  include_directories(${PEER_SRC_DIR}/include)

  set(CMAKE_SWIG_FLAGS "")
  #Oli: changed to R_INCLUDEDIR
  include_directories(${R_INCLUDEDIR})
  include_directories(${R_INCLUDEDIR}/x86_64)
  include_directories(${R_INCLUDEDIR}/i386)

  #OLI: changed to LIBR_SO
  SET(SWIGLINKLIBS ${LIBR_SO} alglib peerlib)

  set_source_files_properties(peer.i PROPERTIES CPLUSPLUS ON)
  set_source_files_properties(peer.i PROPERTIES SWIG_FLAGS "-includeall")
  set_source_files_properties(peer.i PROPERTIES SWIG_FLAGS "-cpperraswarn")
  swig_add_module(peer r peer.i)
  swig_link_libraries(peer ${SWIGLINKLIBS})

  ADD_CUSTOM_COMMAND(
    TARGET peer POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
       ${PEER_SOURCE_DIR}/R/peer 
       ${PEER_SOURCE_DIR}/build/R/peer)

  ADD_CUSTOM_COMMAND(
    TARGET peer POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy
       ${PEER_SOURCE_DIR}/build/R/peerR_wrap.cxx
       ${PEER_SOURCE_DIR}/build/R/peer/src/peerR_wrap.cpp)

  ADD_CUSTOM_COMMAND(
    TARGET peer POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy
       ${PEER_SOURCE_DIR}/build/R/peer.R
       ${PEER_SOURCE_DIR}/build/R/peer/R/peer.R)

  if (BUILD_UNIVERSAL)
    ADD_CUSTOM_COMMAND(
      TARGET peer POST_BUILD
      COMMAND R CMD INSTALL --build ${PEER_SOURCE_DIR}/build/R/peer
      COMMENT "Building and installing the peer R package")
  else (BUILD_UNIVERSAL)
    ADD_CUSTOM_COMMAND(
      TARGET peer POST_BUILD
      COMMAND R CMD INSTALL --no-multiarch --build ${PEER_SOURCE_DIR}/build/R/peer
      COMMENT "Building and installing the peer R package")
  endif (BUILD_UNIVERSAL)

endif(BUILD_R_PACKAGE)

#file(COPY ${CMAKE_CURRENT_BINARY_DIR}/peerR_wrap.cxx 
#     DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/peer/src/peerR_wrap.cpp)
