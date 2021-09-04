cmake_minimum_required(VERSION 3.12)
project(tinycthread C)

set(CMAKE_C_STANDARD 90)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(PACKAGE_NAME TinyCThreads)

enable_testing()

find_package(Threads REQUIRED)

add_library(${PROJECT_NAME} STATIC src/tinycthread.c)
target_link_libraries(${PROJECT_NAME} PRIVATE ${CMAKE_THREAD_LIBS_INIT})
add_library(${PACKAGE_NAME}::Threads ALIAS ${PROJECT_NAME})

include(GNUInstallDirs)
target_include_directories(${PROJECT_NAME}
        PUBLIC
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        )

set_property(TARGET ${PROJECT_NAME} PROPERTY POSITION_INDEPENDENT_CODE True)

set_property(TARGET ${PROJECT_NAME}
        PROPERTY PUBLIC_HEADER
        include/tinycthread.h
        )

set_target_properties(${PROJECT_NAME}
        PROPERTIES
        EXPORT_NAME Threads
        )

install(TARGETS ${PROJECT_NAME}
        EXPORT ${PACKAGE_NAME}Config

        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        )

install(EXPORT ${PACKAGE_NAME}Config
        NAMESPACE ${PACKAGE_NAME}::
        DESTINATION "${CMAKE_INSTALL_DATADIR}/cmake"
        )


export(TARGETS ${PROJECT_NAME}
        NAMESPACE ${PACKAGE_NAME}::
        FILE "${CMAKE_CURRENT_BINARY_DIR}/${PACKAGE_NAME}Config.cmake"
        )

option(TINYCTHREAD_DISABLE_TESTS "Disable TinyCThread unit tests")

if (NOT TINYCTHREAD_DISABLE_TESTS)
    add_executable(test-tinycthread "${CMAKE_CURRENT_SOURCE_DIR}/test/test.c")
    target_link_libraries(test-${PROJECT_NAME} ${PROJECT_NAME})

    add_test(NAME ${PROJECT_NAME}
            COMMAND $<TARGET_FILE:test-${PROJECT_NAME}>)
endif ()
