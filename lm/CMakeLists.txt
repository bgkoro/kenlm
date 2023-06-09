# Explicitly list the source files for this subdirectory
#
# If you add any source files to this subdirectory
#    that should be included in the kenlm library,
#        (this excludes any unit test files)
#    you should add them to the following list:
set(KENLM_LM_SOURCE
	bhiksha.cc
	binary_format.cc
	config.cc
	lm_exception.cc
	model.cc
	quantize.cc
	read_arpa.cc
	search_hashed.cc
	search_trie.cc
	sizes.cc
	trie.cc
	trie_sort.cc
	value_build.cc
	virtual_interface.cc
	vocab.cc
)


# Group these objects together for later use.
#
# Given add_library(foo OBJECT ${my_foo_sources}),
# refer to these objects as $<TARGET_OBJECTS:foo>
#
add_subdirectory(common)

add_library(kenlm ${KENLM_LM_SOURCE} ${KENLM_LM_COMMON_SOURCE})
set_target_properties(kenlm PROPERTIES POSITION_INDEPENDENT_CODE ON)
target_link_libraries(kenlm PUBLIC kenlm_util Threads::Threads)
# Since headers are relative to `include/kenlm` at install time, not just `include`
target_include_directories(kenlm PUBLIC $<INSTALL_INTERFACE:include/kenlm>)

target_compile_definitions(kenlm PUBLIC -DKENLM_MAX_ORDER=${KENLM_MAX_ORDER})

# This directory has children that need to be processed
add_subdirectory(builder)
add_subdirectory(filter)
add_subdirectory(interpolate)

# Explicitly list the executable files to be compiled
set(EXE_LIST
  query
  fragment
  build_binary
  kenlm_benchmark
)

set(LM_LIBS kenlm kenlm_util Threads::Threads)

install(
  TARGETS kenlm
  EXPORT kenlmTargets
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  INCLUDES DESTINATION include
)

AddExes(EXES ${EXE_LIST}
        LIBRARIES ${LM_LIBS})

if(BUILD_TESTING)

  set(KENLM_BOOST_TESTS_LIST left_test partial_test)
  AddTests(TESTS ${KENLM_BOOST_TESTS_LIST}
           LIBRARIES ${LM_LIBS}
           TEST_ARGS ${CMAKE_CURRENT_SOURCE_DIR}/test.arpa)

  # model_test requires an extra command line parameter
  KenLMAddTest(TEST model_test
               LIBRARIES ${LM_LIBS}
               TEST_ARGS ${CMAKE_CURRENT_SOURCE_DIR}/test.arpa
                         ${CMAKE_CURRENT_SOURCE_DIR}/test_nounk.arpa)
endif()
