#
# configure/build libipaca tests
#
##############################################################################
# check if have valgrind
##############################################################################
SET(HAVE_VALGRIND TRUE)
FIND_PROGRAM(VALGRIND_EXECUTABLE valgrind)
IF(VALGRIND_EXECUTABLE STREQUAL "VALGRIND_EXECUTABLE-NOTFOUND")
    MESSAGE(STATUS "WARNING: Could not find valgrind. Will NOT build memory tests.")
    SET(HAVE_VALGRIND FALSE)
ENDIF(VALGRIND_EXECUTABLE STREQUAL "VALGRIND_EXECUTABLE-NOTFOUND")

##############################################################################
# macro definitions
##############################################################################
MACRO(ADD_LIBIPACA_TEST name exe src)
    STRING(REGEX REPLACE "test_([^ ]+).*" "\\1" test "${exe}" )
    # build the test
    ADD_EXECUTABLE(${exe} ${src})
    
    SET(TEST_LIBS  
        ipaca
        #${Boost_LIBRARIES}
    )

    TARGET_LINK_LIBRARIES(${exe} ${TEST_LIBS})
    # add test to global list of unit test
    ADD_TEST(${name} ${exe})
    MESSAGE(STATUS "Adding test for ${name}: ${exe}.")
    # add target for the test
    STRING(REGEX REPLACE "test_([^ ]+).*" "unit_\\1" unittest_target "${exe}" )
    ADD_CUSTOM_TARGET(${unittest_target} COMMAND ${CMAKE_CURRENT_BINARY_DIR}/${exe})
    IF (HAVE_VALGRIND) 
        #add memory test using valgrind
        STRING(REGEX REPLACE "test_([^ ]+).*" "mem_\\1" memtest_name "${exe}" )
        LIST(APPEND memtest_names ${memtest_name})
        LIST(APPEND exe_names ${exe})
        #Add target for the memory test
        ADD_CUSTOM_TARGET(${memtest_name} COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/memcheck.py ${CMAKE_CURRENT_BINARY_DIR}/${exe})
    ENDIF (HAVE_VALGRIND) 
ENDMACRO(ADD_LIBIPACA_TEST name exe src)

# include directories
INCLUDE_DIRECTORIES(
    ${LIBIPACA_SOURCE_DIR}/test/include
    ${LIBIPACA_SOURCE_DIR}/include
    ${LIBIPACA_BINARY_DIR}/include
    ${LIBIPACA_BINARY_DIR}/test/include
    ${CMAKE_CURRENT_SOURCE_DIR}   
)

#### Sources
SET(SRCS_MERCURY7 Mercury7-test.cpp)
SET(SRCS_MERCURY7IMPL Mercury7Impl-test.cpp)
SET(SRCS_STOICHIOMETRY Stoichiometry-test.cpp)

#### Tests
ADD_LIBIPACA_TEST("Mercury7" test_mercury7 ${SRCS_MERCURY7})
ADD_LIBIPACA_TEST("Mercury7Impl" test_mercury7impl ${SRCS_MERCURY7IMPL})
ADD_LIBIPACA_TEST("Stoichiometry" test_stoichiometry ${SRCS_STOICHIOMETRY})

LIST(LENGTH memtest_names numtests)
IF(numtests GREATER 0)
    MATH(EXPR numtests ${numtests}-1)
    FOREACH(i RANGE 0 ${numtests})
        LIST(GET memtest_names ${i} memtest_name)
        LIST(GET exe_names ${i} exe)
        MESSAGE(STATUS "Adding memory test for ${memtest_name}: ${exe}.")
        ADD_TEST(${memtest_name}
            ${CMAKE_CURRENT_SOURCE_DIR}/memtest.py ${CMAKE_CURRENT_BINARY_DIR}/${exe} ${CMAKE_BINARY_DIR})
    ENDFOREACH(i RANGE 0 ${numtests}-1)
ENDIF(numtests GREATER 0)

