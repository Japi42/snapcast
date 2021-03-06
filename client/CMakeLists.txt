set(CLIENT_SOURCES
    client_connection.cpp
    controller.cpp
    snapclient.cpp
    stream.cpp
    time_provider.cpp
    decoder/pcm_decoder.cpp
    player/player.cpp)

set(CLIENT_LIBRARIES ${CMAKE_THREAD_LIBS_INIT} ${ATOMIC_LIBRARY} common)

set(CLIENT_INCLUDE
    ${Boost_INCLUDE_DIR}
    ${CMAKE_SOURCE_DIR}/client
    ${CMAKE_SOURCE_DIR}/common
    ${ASIO_INCLUDE_DIRS}
    ${POPL_INCLUDE_DIRS})


if(MACOSX)
    # Bonjour
    if (BONJOUR_FOUND)
        list(APPEND CLIENT_SOURCES browseZeroConf/browse_bonjour.cpp)
    endif (BONJOUR_FOUND)

    # CoreAudio
    add_definitions(-DHAS_COREAUDIO)
    list(APPEND CLIENT_SOURCES player/coreaudio_player.cpp)
    find_library(COREAUDIO_LIB CoreAudio)
    find_library(COREFOUNDATION_LIB CoreFoundation)
    find_library(AUDIOTOOLBOX_LIB AudioToolbox)
    list(APPEND CLIENT_LIBRARIES ${COREAUDIO_LIB} ${COREFOUNDATION_LIB} ${AUDIOTOOLBOX_LIB})
elseif (WIN32)    
    list(APPEND CLIENT_SOURCES player/wasapi_player.cpp)
    list(APPEND CLIENT_LIBRARIES wsock32 ws2_32 avrt ksuser iphlpapi)
else()
    # Avahi
    if (AVAHI_FOUND)
        list(APPEND CLIENT_SOURCES browseZeroConf/browse_avahi.cpp)
        list(APPEND CLIENT_LIBRARIES ${AVAHI_LIBRARIES})
        list(APPEND CLIENT_INCLUDE ${AVAHI_INCLUDE_DIRS})
    endif (AVAHI_FOUND)

    # ALSA
    if (ALSA_FOUND)
        list(APPEND CLIENT_SOURCES player/alsa_player.cpp)
        list(APPEND CLIENT_LIBRARIES ${ALSA_LIBRARIES})
        list(APPEND CLIENT_INCLUDE ${ALSA_INCLUDE_DIRS})
    endif (ALSA_FOUND)
endif (MACOSX)

#pkg_search_module(SOXR soxr)
find_package(soxr)
if (SOXR_FOUND)
    add_definitions("-DHAS_SOXR")
    list(APPEND CLIENT_LIBRARIES ${SOXR_LIBRARIES})
    list(APPEND CLIENT_INCLUDE ${SOXR_INCLUDE_DIRS})
endif (SOXR_FOUND)

# if OGG then tremor or vorbis
if (OGG_FOUND)
    list(APPEND CLIENT_SOURCES decoder/ogg_decoder.cpp)
    list(APPEND CLIENT_LIBRARIES ${OGG_LIBRARIES})
    list(APPEND CLIENT_INCLUDE ${OGG_INCLUDE_DIRS})
endif (OGG_FOUND)

# Tremor (fixed-point) or libvorbis (floating-point)
if (TREMOR_FOUND)
    list(APPEND CLIENT_LIBRARIES ${TREMOR_LIBRARIES})
    list(APPEND CLIENT_INCLUDE ${TREMOR_INCLUDE_DIRS})
elseif (VORBIS_FOUND)
    list(APPEND CLIENT_LIBRARIES ${VORBIS_LIBRARIES})
    list(APPEND CLIENT_INCLUDE ${VORBIS_INCLUDE_DIRS})
endif (TREMOR_FOUND)

if (FLAC_FOUND)
    list(APPEND CLIENT_SOURCES decoder/flac_decoder.cpp)
    list(APPEND CLIENT_LIBRARIES ${FLAC_LIBRARIES})
    list(APPEND CLIENT_INCLUDE ${FLAC_INCLUDE_DIRS})
endif (FLAC_FOUND)

if (OPUS_FOUND)
    list(APPEND CLIENT_SOURCES decoder/opus_decoder.cpp)
    list(APPEND CLIENT_LIBRARIES ${OPUS_LIBRARIES})
    list(APPEND CLIENT_INCLUDE ${OPUS_INCLUDE_DIRS})
endif (OPUS_FOUND)

include_directories(${CLIENT_INCLUDE})
add_executable(snapclient ${CLIENT_SOURCES})
target_link_libraries(snapclient ${CLIENT_LIBRARIES})

install(TARGETS snapclient COMPONENT client DESTINATION "${CMAKE_INSTALL_BINDIR}")
