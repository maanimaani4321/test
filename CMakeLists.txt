cmake_minimum_required(VERSION 3.15)

project (TeamTalk5SDK)

set (TEAMTALK_ROOT ${CMAKE_CURRENT_LIST_DIR})

option (BUILD_TEAMTALK_CORE "Build TeamTalk core library" ON)
option (BUILD_TEAMTALK_LIBRARIES "Build TeamTalk libraries" OFF)
option (BUILD_TEAMTALK_CLIENTS "Build TeamTalk client examples" OFF)
option (BUILD_TEAMTALK_SERVERS "Build TeamTalk server examples" OFF)
option (BUILD_TEAMTALK_DOCUMENTATION "Build TeamTalk documentation" OFF)

if (BUILD_TEAMTALK_CORE)
  add_subdirectory (Library/TeamTalkLib)
endif()