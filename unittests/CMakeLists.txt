add_custom_target(StandaloneUnitTests)
set_target_properties(StandaloneUnitTests PROPERTIES FOLDER "Standalone Tests")

if (CXX_SUPPORTS_SUGGEST_OVERRIDE_FLAG)
  add_compile_options("-Wno-suggest-override")
endif()

function(add_standalone_unittest test_dirname)
  add_unittest(StandaloneUnitTests ${test_dirname} ${ARGN})
endfunction()

add_subdirectory(Example)
