cmake_minimum_required(VERSION 3.19.0)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VESRION 0.0.${PATCH_VERSION})
set(TARGET "helloworld")
set(LOCAL_COMPILER_C_PATH "/path/to/compiler/C")
set(LOCAL_COMPILER_CXX_PATH "/path/to/compiler/CXX")

option(WITH_BOOST_TEST "Whether to build Boost test" ON)
option(USE_LOCAL_COMPILER "if you want to use local compiler set ON" OFF)

#В сети нашел, что так делать нельзя из - за несовместимости комплияторов
#этот подход либо не будет работать, либо будет работать нестабильно,
#Хотелось бы узнать можно ли как - то с помощью cmake выбирать компиляторы
if(USE_LOCAL_COMPILER)
    set(CMAKE_C_COMPILER LOCAL_COMPILER_C_PATH)
    set(CMAKE_CPP_COMPILER LOCAL_COMPILER_CXX_PATH)
endif()
###

project(${TARGET} VERSION ${PROJECT_VESRION})
configure_file(version.h.in version.h)

add_executable(${TARGET}_cli main.cpp)
add_library(${TARGET}_lib lib.cpp)

set_target_properties(${TARGET}_cli ${TARGET}_lib PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

target_include_directories(${TARGET}_lib
    PRIVATE "${CMAKE_BINARY_DIR}"
)

target_link_libraries(${TARGET}_cli PRIVATE
        ${TARGET}_lib
)

if(WITH_BOOST_TEST)
    find_package(Boost COMPONENTS unit_test_framework REQUIRED)
    add_executable(test_version test_version.cpp)

    set_target_properties(test_version PROPERTIES
        CXX_STANDARD 14
        CXX_STANDARD_REQUIRED ON
    )

    set_target_properties(test_version PROPERTIES
        COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
        INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
    )

    target_link_libraries(test_version
        ${Boost_LIBRARIES} # не понимаю откуда cmake знает про эту переменную
        ${TARGET}_lib
    )
endif()

if (MSVC)
    target_compile_options(${TARGET}_cli PRIVATE
        /W4
    )
    target_compile_options(${TARGET}_lib PRIVATE
        /W4
    )
    if(WITH_BOOST_TEST)
        target_compile_options(test_version PRIVATE
            /W4
        )
    endif()
else ()
    add_compile_options(-Os -g3)
endif()

install(TARGETS ${TARGET}_cli RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT example@example.com)
include(CPack)

if(WITH_BOOST_TEST)
    enable_testing()
    add_test(test_version test_version)
endif()
