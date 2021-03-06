
set(kcrash_SRCS
  kcrash.cpp
  coreconfig.cpp
  ${kcrash_QM_LOADER}
)

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/config-kcrash.h.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/config-kcrash.h
)

add_library(KF5Crash ${kcrash_SRCS})
generate_export_header(KF5Crash BASE_NAME KCrash)
add_library(KF5::Crash ALIAS KF5Crash)

target_include_directories(KF5Crash INTERFACE "$<INSTALL_INTERFACE:${KDE_INSTALL_INCLUDEDIR_KF5}/KCrash>")

target_link_libraries(KF5Crash PUBLIC Qt5::Core)
target_link_libraries(KF5Crash PRIVATE KF5::CoreAddons KF5::WindowSystem)


if(${X11_FOUND})
  find_package(Qt5X11Extras ${REQUIRED_QT_VERSION} REQUIRED NO_MODULE)
  target_link_libraries(KF5Crash PRIVATE Qt5::X11Extras ${X11_LIBRARIES})
  target_include_directories(KF5Crash PRIVATE ${X11_X11_INCLUDE_PATH})
endif()

set_target_properties(KF5Crash PROPERTIES VERSION   ${KCrash_VERSION_STRING}
                                          SOVERSION ${KCrash_SOVERSION}
                                          EXPORT_NAME Crash
)

ecm_generate_headers(KCrash_HEADERS
  HEADER_NAMES
  KCrash

  REQUIRED_HEADERS KCrash_HEADERS
)

install(TARGETS KF5Crash EXPORT KF5CrashTargets ${KF5_INSTALL_TARGETS_DEFAULT_ARGS})

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/kcrash_export.h
  ${KCrash_HEADERS}
  DESTINATION ${KDE_INSTALL_INCLUDEDIR_KF5}/KCrash COMPONENT Devel
)

if(BUILD_QCH)
    ecm_add_qch(
        KF5Crash_QCH
        NAME KCrash
        BASE_NAME KF5Crash
        VERSION ${KF5_VERSION}
        ORG_DOMAIN org.kde
        SOURCES # using only public headers, to cover only public API
            ${KCrash_HEADERS}
        MD_MAINPAGE "${CMAKE_SOURCE_DIR}/README.md"
        LINK_QCHS
            Qt5Core_QCH
        BLANK_MACROS
            KCRASH_EXPORT
            KCRASH_DEPRECATED
            KCRASH_DEPRECATED_EXPORT
        TAGFILE_INSTALL_DESTINATION ${KDE_INSTALL_QTQCHDIR}
        QCH_INSTALL_DESTINATION ${KDE_INSTALL_QTQCHDIR}
        COMPONENT Devel
    )
endif()

include(ECMGeneratePriFile)
ecm_generate_pri_file(BASE_NAME KCrash LIB_NAME KF5Crash DEPS "core" FILENAME_VAR PRI_FILENAME INCLUDE_INSTALL_DIR ${KDE_INSTALL_INCLUDEDIR_KF5}/KCrash)
install(FILES ${PRI_FILENAME} DESTINATION ${ECM_MKSPECS_INSTALL_DIR})
