cmake_minimum_required(VERSION 3.22.1)
project(tbert LANGUAGES CXX C)

set(TARGET tbert)
set(CMAKE_C_STANDARD   11)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED true)
set(CMAKE_C_STANDARD_REQUIRED true)
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

#set(CMAKE_VERBOSE_MAKEFILE ON)

add_library(tbert SHARED MODULE library.cc)
set_target_properties(tbert PROPERTIES POSITION_INDEPENDENT_CODE ON)

#add_compile_options(-g -O0 -DDEBUG -Wall -Wextra -Wconversion)

include(${CMAKE_CURRENT_LIST_DIR}/FindTCL.cmake)

if (NAVISERVER)
    include_directories(${NAVISERVER}/include)
    target_link_directories(tbert PRIVATE ${NAVISERVER}/lib)
    target_link_libraries(tbert PRIVATE bert ggml ${TCL_LIBRARY} nsd)
    target_compile_options(tbert PRIVATE -DUSE_NAVISERVER)
    install(TARGETS ${TARGET}
            LIBRARY DESTINATION ${NAVISERVER}/bin
    )
else ()
    link_directories(bert.cpp/build bert.cpp/build/ggml/src)
    target_link_libraries(tbert PRIVATE bert ggml ${TCL_LIBRARY})
    get_filename_component(TCL_LIBRARY_PATH "${TCL_LIBRARY}" PATH)
    if (TCL_LIBRARY_PATH)
        install(TARGETS ${TARGET}
                LIBRARY DESTINATION ${TCL_LIBRARY_PATH}/${TARGET}
        )
        install(FILES pkgIndex.tcl
                DESTINATION ${TCL_LIBRARY_PATH}/${TARGET}
        )
    endif()
endif()

