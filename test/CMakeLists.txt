configure_lit_site_cfg(
        ${CMAKE_CURRENT_SOURCE_DIR}/lit.site.cfg.py.in
        ${CMAKE_CURRENT_BINARY_DIR}/lit.site.cfg.py
        MAIN_CONFIG
        ${CMAKE_CURRENT_SOURCE_DIR}/lit.cfg.py
)
configure_lit_site_cfg(
  ${CMAKE_CURRENT_SOURCE_DIR}/Unit/lit.site.cfg.py.in
  ${CMAKE_CURRENT_BINARY_DIR}/Unit/lit.site.cfg.py
  MAIN_CONFIG
  ${CMAKE_CURRENT_SOURCE_DIR}/Unit/lit.cfg.py
)

set(STANDALONE_OPT_TEST_DEPENDS
        FileCheck count not
        standalone-opt
        )

if (STANDALONE_GTEST_AVAILABLE)
  message(WARNING "HELLO")
  list(APPEND STANDALONE_OPT_TEST_DEPENDS StandaloneUnitTests)
endif()

add_lit_testsuite(check-standalone-opt "Running the standalone-opt regression tests"
        ${CMAKE_CURRENT_BINARY_DIR}
        DEPENDS ${STANDALONE_OPT_TEST_DEPENDS}
        )
set_target_properties(check-standalone-opt PROPERTIES FOLDER "Tests")

add_lit_testsuites(STANDALONE_OPT ${CMAKE_CURRENT_SOURCE_DIR} DEPENDS ${STANDALONE_OPT_TEST_DEPENDS})
