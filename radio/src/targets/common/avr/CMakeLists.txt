option(AUDIO "Audio support" OFF)
option(VOICE "Voice support" OFF)
option(GRAPHICS "Additional graphics" ON)
option(BOLD "Bold font" ON)
option(BATTGRAPH "Battery graph" OFF)
option(HAPTIC "Haptic support" OFF)

set(TELEMETRY_TYPES NONE/MAVLINK/ARDUPILOT/FRSKY/FRSKY_SPORT/TELEMETREZ/NMEA)
set(TELEMETRY "NONE" CACHE STRING "Telemetry Type (${TELEMETRY_TYPES})")
set_property(CACHE TELEMETRY PROPERTY STRINGS ${TELEMETRY_TYPES})

set(TTS "EN" CACHE STRING "TTS language, one of: ${TTS_LANGUAGES}")
set_property(CACHE TTS PROPERTY STRINGS ${TTS_LANGUAGES})

set(LCD_TYPES DEFAULT ST7565P ST7565R ERC12864FSF ST7920 KS108)
set(LCD "DEFAULT" CACHE STRING "LCD type (${LCD_TYPES})")
set_property(CACHE LCD PROPERTY STRINGS ${LCD_TYPES})

set(DSM2 "NO" CACHE STRING "DSM2 support/interface (NO/PPM/SERIAL)")
set_property(CACHE DSM2 PROPERTY STRINGS NO PPM SERIAL)

set(LUA NO)
set(PULSES_SRC pulses_avr.cpp)
set(SRC ${SRC} main_avr.cpp)
set(FIRMWARE_SRC
  ${FIRMWARE_SRC}
  targets/common/avr/adc_driver.cpp
  targets/common/avr/eeprom_driver.cpp
  )
add_definitions(-DLCD_${LCD})
if(DSM2)
  add_definitions(-DDSM2 -DDSM2_${DSM2})
endif()
if(SDCARD)
  set(FIRMWARE_SRC ${FIRMWARE_SRC} targets/gruvin9x/diskio.cpp)
endif()
if(AUDIO)
  add_definitions(-DAUDIO)
  set(SRC ${SRC} audio_avr.cpp)
  set(BUZZER OFF)
else()
  set(BUZZER ON)
endif()
if(BUZZER)
  add_definitions(-DBUZZER)
  set(SRC ${SRC} buzzer.cpp)
endif()
if(VOICE)
  add_definitions(-DVOICE)
  set(TARGET_SRC ${TARGET_SRC} voice.cpp)
  foreach(LANGUAGE ${TTS_LANGUAGES})
    if(TTS STREQUAL ${LANGUAGE})
      add_definitions(-DTTS_${LANGUAGE})
      string(TOLOWER ${LANGUAGE} lang_lower)
      set(SRC ${SRC} translations/tts_${lang_lower}.cpp)
    endif()
  endforeach()
endif()
if(GRAPHICS)
  add_definitions(-DGRAPHICS)
endif()
if(BOLD)
  add_definitions(-DBOLD_FONT)
endif()
if(BATTGRAPH)
  add_definitions(-DBATTGRAPH)
endif()
if(HAPTIC)
  add_definitions(-DHAPTIC)
  set(SRC ${SRC} haptic.cpp)
endif()
if(${TELEMETRY} STREQUAL OFF)
  add_definitions(-DTELEMETRY_NONE)
elseif(TELEMETRY STREQUAL ARDUPILOT)
  add_definitions(-DTELEMETRY_ARDUPILOT)
  set(SRC ${SRC} telemetry/ardupilot.cpp)
elseif(TELEMETRY STREQUAL NMEA)
  add_definitions(-DTELEMETRY_NMEA)
  set(SRC ${SRC} telemetry/nmea.cpp)
elseif(TELEMETRY STREQUAL JETI)
  add_definitions(-DTELEMETRY_JETI)
  set(SRC ${SRC} telemetry/jeti.cpp)
elseif(TELEMETRY STREQUAL MAVLINK)
  add_definitions(-DTELEMETRY_MAVLINK)
  include_directories(${THIRDPARTY_DIR} gui/${GUI_DIR} targets/common/avr)
  set(FIRMWARE_SRC
    ${FIRMWARE_SRC}
    targets/common/avr/telemetry_driver.cpp
    )
  set(SRC ${SRC} telemetry/mavlink.cpp)
  set(GUI_SRC ${GUI_SRC} view_mavlink.cpp)
  math(EXPR EEPROM_VARIANT ${EEPROM_VARIANT}+${MAVLINK_VARIANT})
elseif(TELEMETRY STREQUAL TELEMETREZ)
  add_definitions(-DTELEMETRY_TELEMETREZ)
  set(SRC ${SRC} telemetry/jeti.cpp)
elseif(TELEMETRY STREQUAL FRSKY_SPORT)
  add_definitions(-DTELEMETRY_FRSKY_SPORT)
  set(SRC ${SRC} crc16.cpp telemetry/frsky_sport.cpp)
endif()
if(TELEMETRY STREQUAL FRSKY OR TELEMETRY STREQUAL FRSKY_SPORT OR TELEMETRY STREQUAL TELEMETREZ)
  option(FRSKY_HUB "FrSky Hub support" ON)
  option(WS_HOW_HIGH "Winged Shadow sensors support" ON)
  option(GAUGES "Telemetry gauges" OFF)
  option(GPS "GPS support" ON)
  option(VARIO "Vario support" ON)
  add_definitions(-DTELEMETRY_FRSKY)
  set(SRC ${SRC} telemetry/telemetry.cpp telemetry/telemetry_holders.cpp telemetry/frsky.cpp telemetry/frsky_d.cpp)
  set(FIRMWARE_SRC
    ${FIRMWARE_SRC}
    targets/common/avr/telemetry_driver.cpp
    )
  set(GUI_SRC ${GUI_SRC} view_telemetry.cpp)
  if(FRSKY_HUB)
    add_definitions(-DFRSKY_HUB)
  endif()
  if(WS_HOW_HIGH)
    add_definitions(-DWS_HOW_HIGH)
  endif()
  if(GAUGES)
    add_definitions(-DGAUGES)
  endif()
  if(GPS)
    add_definitions(-DGPS)
  endif()
  if(VARIO)
    add_definitions(-DVARIO)
    set(SRC ${SRC} vario.cpp)
  endif()
  math(EXPR EEPROM_VARIANT ${EEPROM_VARIANT}+${FRSKY_VARIANT})
endif()
add_definitions(-DTIMERS=2)
if(EEPROM_VARIANT_NEEDED)
  add_definitions(-DEEPROM_VARIANT=${EEPROM_VARIANT})
endif()

