project (TeamTalk5Lib LANGUAGES CXX)

set (CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

include (FeatureSummary)

set (TEAMTALKLIB_ROOT ${CMAKE_CURRENT_LIST_DIR})

set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DDEBUG -D_DEBUG")
set (CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -DDEBUG -D_DEBUG")

set (CMAKE_CXX_STANDARD 23)
set (CMAKE_CXX_STANDARD_REQUIRED ON)

if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  option (STATICCPP "Link static C and C++ libraries" ON)
  if (STATICCPP)
    set (LINK_LIBS -static-libgcc -static-libstdc++)
  endif()
endif()

if (MSVC)
  set (COMPILE_FLAGS -DUNICODE -D_UNICODE
    -D_CRT_NONSTDC_NO_WARNINGS
    -D_CRT_SECURE_NO_WARNINGS
    -D_CRT_SECURE_NO_DEPRECATE
    -D_CRT_NONSTDC_NO_DEPRECATE
    -D_WINSOCK_DEPRECATED_NO_WARNINGS)
  set (PLATFORMTOOLSET "/property:PlatformToolset=v145")
else()
  set (COMPILE_FLAGS -Wno-deprecated-declarations)
endif()

if (${CMAKE_SYSTEM_NAME} MATCHES "Android")
  set (DLL_INSTALL_DIR Library/TeamTalk_DLL/${CMAKE_ANDROID_ARCH_ABI})
else()
  set (DLL_INSTALL_DIR Library/TeamTalk_DLL)
endif()

function(set_output_dir target dir)
  set_target_properties (${target} PROPERTIES
    LIBRARY_OUTPUT_DIRECTORY ${dir}
    LIBRARY_OUTPUT_DIRECTORY_DEBUG ${dir}
    LIBRARY_OUTPUT_DIRECTORY_RELEASE ${dir}
    RUNTIME_OUTPUT_DIRECTORY ${dir}
    RUNTIME_OUTPUT_DIRECTORY_DEBUG ${dir}
    RUNTIME_OUTPUT_DIRECTORY_RELEASE ${dir}
    ARCHIVE_OUTPUT_DIRECTORY ${dir}
    ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${dir}
    ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${dir})
endfunction()

option (FEATURE_LIBVPX "Build TeamTalk DLL with libVPX support" ON)
option (FEATURE_OGG "Build TeamTalk DLL with libOgg support" ON)
option (FEATURE_OPUS "Build TeamTalk DLL with OPUS codec support" ON)
option (FEATURE_OPUSTOOLS "Build TeamTalk DLL with Opus-tools support" ON)
option (FEATURE_SPEEX "Build TeamTalk DLL with Speex codec support" ON)
option (FEATURE_SPEEXDSP "Build TeamTalk DLL with SpeexDSP audio processing support" ON)
option (FEATURE_WEBRTC "Build TeamTalk DLL with WebRTC audio processing support" ON)
option (FEATURE_MSDMO "Build TeamTalk DLL with Microsoft DirectX Media Objects (DMO) Resampler support" ON)
option (FEATURE_MEDIAFOUNDATION "Build TeamTalk DLL with Media Foundation Streaming support" ON)
option (FEATURE_FFMPEG "Build TeamTalk DLL with FFmpeg support" ON)

if (${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  option (FEATURE_PORTAUDIO "Build using PortAudio as sound system" ON)
endif()

add_subdirectory (build)

list (APPEND CMAKE_MODULE_PATH "${TEAMTALKLIB_ROOT}/build")

include (ttclient)
include (ttdll)

######################
# TeamTalk5.dll & .lib
######################

option (BUILD_TEAMTALK_LIBRARY_DLL "Build TeamTalk5 shared library" ON)
if (BUILD_TEAMTALK_LIBRARY_DLL)
  add_library (TeamTalk5DLL SHARED ${TTCLIENTDLL_SOURCES} ${TTCLIENTDLL_HEADERS})
  target_include_directories (TeamTalk5DLL PRIVATE ${TTCLIENTDLL_INCLUDE_DIR} INTERFACE ${TEAMTALK_ROOT}/Library/TeamTalk_DLL)
  target_compile_options (TeamTalk5DLL PRIVATE ${TTCLIENTDLL_COMPILE_FLAGS} ${COMPILE_FLAGS})
  target_link_libraries (TeamTalk5DLL PRIVATE ${TTCLIENTDLL_LINK_FLAGS} ${LINK_LIBS})
  set_target_properties (TeamTalk5DLL PROPERTIES OUTPUT_NAME TeamTalk5)

  if (MSVC AND ${CMAKE_SIZEOF_VOID_P} EQUAL 4)
    target_link_options (TeamTalk5DLL PRIVATE /NODEFAULTLIB:MSVCRT /NODEFAULTLIB:vcruntime.lib)
  endif()

  set_output_dir(TeamTalk5DLL ${TEAMTALK_ROOT}/Library/TeamTalk_DLL)

  install (FILES ${CMAKE_CURRENT_LIST_DIR}/../TeamTalk_DLL/TeamTalk.h DESTINATION Library/TeamTalk_DLL)
  install (TARGETS TeamTalk5DLL DESTINATION ${DLL_INSTALL_DIR})
endif()

option (BUILD_TEAMTALK_LIBRARY_LIB "Build TeamTalk5 static library" ON)
if (BUILD_TEAMTALK_LIBRARY_LIB)
  add_library (TeamTalk5Lib STATIC ${TTCLIENTDLL_SOURCES} ${TTCLIENTDLL_HEADERS})
  target_include_directories (TeamTalk5Lib PRIVATE ${TTCLIENTDLL_INCLUDE_DIR} INTERFACE ${TEAMTALK_ROOT}/Library/TeamTalk_DLL)
  target_compile_options (TeamTalk5Lib PRIVATE ${TTCLIENTDLL_COMPILE_FLAGS} ${COMPILE_FLAGS})
  target_link_libraries (TeamTalk5Lib PRIVATE ${TTCLIENTDLL_LINK_FLAGS} ${LINK_LIBS})
  set_target_properties (TeamTalk5Lib PROPERTIES OUTPUT_NAME TeamTalk5)

  set_output_dir(TeamTalk5Lib ${TEAMTALK_ROOT}/Library/TeamTalk_DLL)
  install (TARGETS TeamTalk5Lib DESTINATION ${DLL_INSTALL_DIR})
endif()