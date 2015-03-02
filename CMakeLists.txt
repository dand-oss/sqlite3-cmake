PROJECT(sqlite3 C CXX)
cmake_minimum_required(VERSION 2.8)

###############
# ASI default settings
list(APPEND CMAKE_MODULE_PATH "$ENV{ASI}/cmake/Modules")
include(asv_cmake_defaults)
asv_cmake_defaults( )
###############

include_directories(${CMAKE_SOURCE_DIR}/src)
set(exename sqlite)
set(libname sqlite3)
if(UNIX)
  add_library(${libname} SHARED src/sqlite3.c src/sqlite3.h src/sqlite3ext.h)
  add_executable(${exename} src/sqlite3.c src/shell.c src/sqlite3.h src/sqlite3ext.h)
  target_link_libraries(${exename} -ldl -lpthread -lm)
  set_target_properties(${exename} PROPERTIES OUTPUT_NAME ${libname})
else()
  add_library(${libname} STATIC src/sqlite3.c src/sqlite3.h src/sqlite3ext.h)
  # later set_target_properties(${libname} PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS ON)
  install_static_pdb(${libname}) # ASV
  add_executable(${exename} src/sqlite3.c src/shell.c src/sqlite3.h src/sqlite3ext.h)
endif()

add_definitions(-DSQLITE_ENABLE_RTREE)
add_definitions(-DSQLITE_ENABLE_FTS4)
add_definitions(-DSQLITE_ENABLE_FTS5)
add_definitions(-DSQLITE_ENABLE_JSON1)
add_definitions(-DSQLITE_ENABLE_RBU)
add_definitions(-DSQLITE_ENABLE_STAT4)

# Uncomment this for single-threaded variant (faster)
#add_definitions(-DSQLITE_THREADSAFE=0)

if(NOT WIN32)
  include(FindThreads)
  target_link_libraries(sqlite m ${CMAKE_THREAD_LIBS_INIT} ${CMAKE_DL_LIBS})
endif()

install(FILES src/sqlite3.h src/sqlite3ext.h DESTINATION include)
install(TARGETS ${exename} RUNTIME DESTINATION bin)
install(TARGETS ${libname}
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    )
