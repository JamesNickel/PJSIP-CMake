cmake_minimum_required(VERSION 2.8.8)

project(pjproject)

################################################################################
# build options
################################################################################

# miscs
set(CMAKE_VERBOSE_MAKEFILE ON)

# targeted OS
set(TARGET_LINUX  OFF CACHE BOOL "target OS is Linux")

# features
set(WITH_SRTP                   ON  CACHE BOOL "enable SRTP")

set(WITH_AUDIODEV_PORTAUDIO     OFF CACHE BOOL "enable portaudio audio device")   # TODO: test ON
set(WITH_AUDIODEV_WMME          OFF CACHE BOOL "enable WMME audio device")        # TODO: test ON

set(WITH_RESAMPLE_LIBRESAMPLE   OFF CACHE BOOL "enable libresample ressampling")
set(WITH_RESAMPLE_LIBSAMPLERATE OFF CACHE BOOL "enable libsamplerate ressampling")# TODO: test ON

set(WITH_CODEC_SPEEX            OFF CACHE BOOL "enable speex codec")
set(WITH_CODEC_ILBC             OFF CACHE BOOL "enable ilbc codec")
set(WITH_CODEC_GSM              OFF CACHE BOOL "enable gsm codec")
set(WITH_CODEC_G722             OFF CACHE BOOL "enable g722 codec")
set(WITH_CODEC_G7221            OFF CACHE BOOL "enable g7221 codec")
set(WITH_CODEC_L16              OFF CACHE BOOL "enable l16 codec")

set(WITH_AEC_SPEEX              OFF CACHE BOOL "enable speex aec")

# auto-fix incompatibles options
if(NOT WITH_CODEC_SPEEX AND WITH_AEC_SPEEX)
  set(WITH_AEC_SPEEX OFF)
  message(WARNING "disabling WITH_AEC_SPEEX as WITH_CODEC_SPEEX is OFF...")
endif()

################################################################################
# Features specific
################################################################################

if(WITH_SRTP)
  add_definitions(-DPJMEDIA_HAS_SRTP=1)
else()
  add_definitions(-DPJMEDIA_HAS_SRTP=0)
endif()

if(WITH_AUDIODEV_PORTAUDIO)
  add_definitions(-DPJMEDIA_AUDIO_DEV_HAS_PORTAUDIO=1)
else()
  add_definitions(-DPJMEDIA_AUDIO_DEV_HAS_PORTAUDIO=0)
endif()

if(WITH_AUDIODEV_WMME)
  add_definitions(-DPJMEDIA_AUDIO_DEV_HAS_WMME=1)
else()
  add_definitions(-DPJMEDIA_AUDIO_DEV_HAS_WMME=0)
endif()

if(WITH_RESAMPLE_LIBRESAMPLE)
  add_definitions(-DPJMEDIA_RESAMPLE_IMP=PJMEDIA_RESAMPLE_LIBRESAMPLE)
  set(RESAMPLING_NAME "resample")
elseif(WITH_RESAMPLE_LIBSAMPLERATE)
  add_definitions(-DPJMEDIA_RESAMPLE_IMP=PJMEDIA_RESAMPLE_LIBSAMPLERATE)
  set(RESAMPLING_NAME "libsamplerate")
else()
  add_definitions(-DPJMEDIA_RESAMPLE_IMP=PJMEDIA_RESAMPLE_NONE)
  set(RESAMPLING_NAME "none")
endif()

if(WITH_CODEC_SPEEX)
  add_definitions(-DPJMEDIA_HAS_SPEEX_CODEC=1)
else()
  add_definitions(-DPJMEDIA_HAS_SPEEX_CODEC=0)
endif()

if(WITH_CODEC_ILBC)
  add_definitions(-DPJMEDIA_HAS_ILBC_CODEC=1)
else()
  add_definitions(-DPJMEDIA_HAS_ILBC_CODEC=0)
endif()

if(WITH_CODEC_GSM)
  add_definitions(-DPJMEDIA_HAS_GSM_CODEC=1)
else()
  add_definitions(-DPJMEDIA_HAS_GSM_CODEC=0)
endif()

if(WITH_CODEC_G722)
  add_definitions(-DPJMEDIA_HAS_G722_CODEC=1)
else()
  add_definitions(-DPJMEDIA_HAS_G722_CODEC=0)
endif()

if(WITH_CODEC_G7221)
  add_definitions(-DPJMEDIA_HAS_G7221_CODEC=1)
else()
  add_definitions(-DPJMEDIA_HAS_G7221_CODEC=0)
endif()

if(WITH_CODEC_L16)
  add_definitions(-DPJMEDIA_HAS_L16_CODEC=1)
else()
  add_definitions(-DPJMEDIA_HAS_L16_CODEC=0)
endif()

if(WITH_AEC_SPEEX)
  add_definitions(-DPJMEDIA_HAS_SPEEX_AEC=1)
else()
  add_definitions(-DPJMEDIA_HAS_SPEEX_AEC=0)
endif()


################################################################################
# OS specific
################################################################################

# Linux
if(TARGET_LINUX)
  set(CMAKE_SYSTEM_NAME Linux)
  add_definitions(-DPJ_LINUX=1)
  set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -fPIC)
  set(OS_LIBS rt m pthread)
  if(WITH_SRTP)
    set(OS_LIBS ${OS_LIBS} ssl)
  endif()
endif()


################################################################################
# Print summary
################################################################################
message("Target OS:   ${CMAKE_SYSTEM_NAME}")
message("Options:")
message("- SRTP:      ${WITH_SRTP}")
message("- Speex AEC: ${WITH_AEC_SPEEX}")
message("Codecs:")
message("- speex:     ${WITH_CODEC_SPEEX}")
message("- gsm:       ${WITH_CODEC_GSM}")
message("- ilbc:      ${WITH_CODEC_ILBC}")
message("- g722:      ${WITH_CODEC_G722}")
message("- g7221:     ${WITH_CODEC_G7221}")
message("- L16:       ${WITH_CODEC_L16}")
message("Resampling:  ${RESAMPLING_NAME}")


################################################################################
# Common
################################################################################

# sub-directories
set(PJSIP_SUBDIRECTORIES pjlib pjlib-util pjnath pjmedia pjsip)

foreach(SUBDIRECTORY ${PJSIP_SUBDIRECTORIES})
  set(PJSIP_INCLUDES ${PJSIP_INCLUDES} ${SUBDIRECTORY}/include/)
endforeach()
include_directories(${CMAKE_FIND_ROOT_PATH}/usr/include})
include_directories(AFTER SYSTEM . ${PJSIP_INCLUDES} ${CMAKE_BINARY_DIR})

add_subdirectory(third_party/build)
foreach(SUBDIRECTORY ${PJSIP_SUBDIRECTORIES})
  add_subdirectory(${SUBDIRECTORY}/build)
  set(TARGET_OBJECTS_LIST ${TARGET_OBJECTS_LIST} $<TARGET_OBJECTS:${SUBDIRECTORY}>)
endforeach()

# build target
add_library(${PROJECT_NAME} STATIC ${TARGET_OBJECTS_LIST} ${THIRDPARTY_OBJECTS_LIST})
target_link_libraries(${PROJECT_NAME} ${OS_LIBS})

file(GLOB PJSUA-APP_SRC pjsip-apps/src/pjsua/pjsua_app*)
set(PJSUA-APP_SRC ${PJSUA-APP_SRC}
  pjsip-apps/src/pjsua/main.c)
add_executable(pjsua ${PJSUA-APP_SRC})
target_link_libraries(pjsua ${PROJECT_NAME})