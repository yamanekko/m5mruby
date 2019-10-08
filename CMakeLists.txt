set(COMPONENT_ADD_INCLUDEDIRS "")

register_component()

set(MRUBY_DIR "${COMPONENT_PATH}/mruby")

set(MRUBY_INCLUDES ${COMPONENT_PATH}/../../build/config
    ${COMPONENT_PATH}/../m5stack/src
    ${IDF_PATH}/components/bt/bluedroid/api/include/api
    ${COMPONENT_PATH}/../arduino/variants/esp32
    ${COMPONENT_PATH}/../arduino/cores/esp32
    ${COMPONENT_PATH}/../arduino/libraries/ArduinoOTA/src
    ${COMPONENT_PATH}/../arduino/libraries/AsyncUDP/src
    ${COMPONENT_PATH}/../arduino/libraries/AzureIoT/src
    ${COMPONENT_PATH}/../arduino/libraries/BLE/src
    ${COMPONENT_PATH}/../arduino/libraries/BluetoothSerial/src
    ${COMPONENT_PATH}/../arduino/libraries/DNSServer/src
    ${COMPONENT_PATH}/../arduino/libraries/EEPROM/src
    ${COMPONENT_PATH}/../arduino/libraries/ESP32/src
    ${COMPONENT_PATH}/../arduino/libraries/ESPmDNS/src
    ${COMPONENT_PATH}/../arduino/libraries/FFat/src
    ${COMPONENT_PATH}/../arduino/libraries/FS/src
    ${COMPONENT_PATH}/../arduino/libraries/HTTPClient/src
    ${COMPONENT_PATH}/../arduino/libraries/HTTPUpdate/src
    ${COMPONENT_PATH}/../arduino/libraries/NetBIOS/src
    ${COMPONENT_PATH}/../arduino/libraries/Preferences/src
    ${COMPONENT_PATH}/../arduino/libraries/SD_MMC/src
    ${COMPONENT_PATH}/../arduino/libraries/SD/src
    ${COMPONENT_PATH}/../arduino/libraries/SimpleBLE/src
    ${COMPONENT_PATH}/../arduino/libraries/SPIFFS/src
    ${COMPONENT_PATH}/../arduino/libraries/SPI/src
    ${COMPONENT_PATH}/../arduino/libraries/Ticker/src
    ${COMPONENT_PATH}/../arduino/libraries/Update/src
    ${COMPONENT_PATH}/../arduino/libraries/WebServer/src
    ${COMPONENT_PATH}/../arduino/libraries/WiFiClientSecure/src
    ${COMPONENT_PATH}/../arduino/libraries/WiFi/src
    ${COMPONENT_PATH}/../arduino/libraries/Wire/src
    ${IDF_PATH}/components/spi_flash/include
    ${IDF_PATH}/components/mbedtls/port/include
    ${IDF_PATH}/components/mbedtls/mbedtls/include
    ${IDF_PATH}/components/lwip/include/apps
    ${IDF_PATH}/components/lwip/lwip/src/include
    ${IDF_PATH}/components/lwip/port/esp32/include
    ${IDF_PATH}/components/lwip/port/esp32/include/arch
    ${IDF_PATH}/components/lwip/include_compat
    ${IDF_PATH}/components/vfs/include
    ${IDF_PATH}/components/mdns/include
    ${IDF_PATH}/components/console
    ${IDF_PATH}/components/tcpip_adapter/include
    ${IDF_PATH}/components/ethernet/include
    ${IDF_PATH}/components/esp32/include
    ${IDF_PATH}/components/driver/include
    ${IDF_PATH}/components/esp_ringbuf/include
    ${IDF_PATH}/components/esp_event/include
    ${IDF_PATH}/components/log/include
    ${IDF_PATH}/components/newlib/platform_include
    ${IDF_PATH}/components/newlib/include
    ${IDF_PATH}/components/freertos/include
    ${IDF_PATH}/components/app_trace/include
    ${IDF_PATH}/components/heap/include
    ${IDF_PATH}/components/soc/esp32/include
    ${IDF_PATH}/components/soc/include)

#
# libmruby.a target
#
set(LIBMRUBY_FILE "${COMPONENT_PATH}/mruby/build/esp32/lib/libmruby.a")

add_custom_command(
  OUTPUT ${LIBMRUBY_FILE}
  COMMAND rake "COMPONENT_INCLUDES=${MRUBY_INCLUDES}" "CC=${CMAKE_C_COMPILER}" "LD=${CMAKE_LINKER}" "AR=${CMAKE_AR}" "TARGET_INCLUDES=$<TARGET_PROPERTY:${COMPONENT_NAME},INTERFACE_INCLUDE_DIRECTORIES>" "MRUBY_CONFIG=${PROJECT_PATH}/build_config.rb"
  COMMENT "building mruby..."
  WORKING_DIRECTORY ${MRUBY_DIR}
  VERBATIM
)

add_library(libmruby STATIC IMPORTED)
set_target_properties(libmruby PROPERTIES IMPORTED_LOCATION ${LIBMRUBY_FILE})
set_target_properties(libmruby PROPERTIES INTERFACE_INCLUDE_DIRECTORIES "${COMPONENT_PATH}/mruby/include")

add_custom_target(mruby DEPENDS ${LIBMRUBY_FILE})
add_dependencies(${COMPONENT_NAME} mruby)

target_link_libraries(${COMPONENT_NAME} INTERFACE libmruby)

#
# add clean files
#
set_property(DIRECTORY "${COMPONENT_PATH}" APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES "${MRUBY_DIR}/build")
