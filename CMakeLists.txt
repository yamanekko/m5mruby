set(COMPONENT_ADD_INCLUDEDIRS "mruby/include")

register_component()

set(MRUBY_DIR "${COMPONENT_PATH}/mruby")

#
# libmruby.a
#
set(LIBMRUBY_FILE "${COMPONENT_PATH}/mruby/build/esp32/lib/libmruby.a")

# search build_config.rb
if(EXISTS ${PROJECT_PATH}/esp32_build_config.rb)
  set(MRUBY_CONFIG ${PROJECT_PATH}/esp32_build_config.rb)
elseif(EXISTS ${PROJECT_PATH}/build_config.rb)
  set(MRUBY_CONFIG ${PROJECT_PATH}/build_config.rb)
else()
  set(MRUBY_CONFIG ${COMPONENT_PATH}/esp32_build_config.rb)
endif()

# build libmruby.a
add_custom_command(
  OUTPUT ${LIBMRUBY_FILE}
  COMMAND ${CMAKE_COMMAND} -E env "MRUBY_CONFIG=${MRUBY_CONFIG}" "CC=${CMAKE_C_COMPILER}" "LD=${CMAKE_LINKER}" "AR=${CMAKE_AR}" "COMPONENT_INCLUDES=$<TARGET_PROPERTY:${COMPONENT_NAME},INTERFACE_INCLUDE_DIRECTORIES>" rake
  COMMENT "building mruby..."
  WORKING_DIRECTORY ${MRUBY_DIR}
  VERBATIM
)

# libmruby is imported library
add_library(libmruby STATIC IMPORTED)
set_target_properties(libmruby PROPERTIES IMPORTED_LOCATION ${LIBMRUBY_FILE})
set_target_properties(libmruby PROPERTIES INTERFACE_INCLUDE_DIRECTORIES "${COMPONENT_PATH}/mruby/include")
set_target_properties(libmruby PROPERTIES INTERFACE_INCLUDE_DIRECTORIES "${PROJECT_PATH}/build/config")

# use headers of COMPONENT_REQUIRES_COMMON
set(reqs ${COMPONENT_REQUIRES_COMMON})
spaces2list(reqs)
target_link_libraries(libmruby INTERFACE ${reqs})

# use headers of components
file(GLOB com_dirs RELATIVE "${PROJECT_PATH}/components/" "${PROJECT_PATH}/components/*")
list(REMOVE_ITEM com_dirs "${COMPONENT_NAME}")
foreach(com ${com_dirs})
  target_link_libraries(libmruby INTERFACE "${com}")
endforeach()

target_link_libraries(${COMPONENT_NAME} INTERFACE libmruby)

#
# target mruby
#
add_custom_target(mruby DEPENDS ${LIBMRUBY_FILE})
add_dependencies(${COMPONENT_NAME} mruby)

#
# add clean files
#
set_property(DIRECTORY "${COMPONENT_PATH}" APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES "${MRUBY_DIR}/build")
set_property(DIRECTORY "${COMPONENT_PATH}" APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES "${COMPONENT_PATH}/esp32_build_config.rb.lock")
set_property(DIRECTORY "${COMPONENT_PATH}" APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES "${PROJECT_PATH}/esp32_build_config.rb.lock")
set_property(DIRECTORY "${COMPONENT_PATH}" APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES "${PROJECT_PATH}/build_config.rb.lock")
