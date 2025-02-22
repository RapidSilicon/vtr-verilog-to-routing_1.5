cmake_minimum_required(VERSION 3.16)

project("libarchfpga")

file(GLOB_RECURSE READ_ARCH_EXEC_SRC src/main.cpp)
file(GLOB_RECURSE LIB_SOURCES src/*.cpp)
file(GLOB_RECURSE LIB_HEADERS src/*.h)
files_to_dirs(LIB_HEADERS LIB_INCLUDE_DIRS)

#Create the library
add_library(libarchfpga STATIC
    ${LIB_HEADERS}
    ${LIB_SOURCES}
)

target_include_directories(libarchfpga PUBLIC ${LIB_INCLUDE_DIRS})

set_target_properties(libarchfpga PROPERTIES PREFIX "") #Avoid extra 'lib' prefix

#Specify link-time dependancies
target_link_libraries(libarchfpga
                        libvtrutil
                        libpugixml
                        libpugiutil
                        libvtrcapnproto
)

target_compile_definitions(libarchfpga PUBLIC ${INTERCHANGE_SCHEMA_HEADERS})

#Create the test executable
add_executable(read_arch ${READ_ARCH_EXEC_SRC})
target_link_libraries(read_arch libarchfpga)

#Supress IPO link warnings if IPO is enabled
get_target_property(READ_ARCH_USES_IPO read_arch INTERPROCEDURAL_OPTIMIZATION)
if (READ_ARCH_USES_IPO)
    set_property(TARGET read_arch APPEND PROPERTY LINK_FLAGS ${IPO_LINK_WARN_SUPRESS_FLAGS})
endif()

install(TARGETS libarchfpga DESTINATION bin)
install(FILES ${LIB_HEADERS} DESTINATION include/libarchfpga)

# 
# Unit Tests
#
file(GLOB_RECURSE TEST_SOURCES test/*.cpp)
add_executable(test_archfpga ${TEST_SOURCES})
target_link_libraries(test_archfpga Catch2::Catch2WithMain libarchfpga)
