#
# This file is part of the CMaNGOS Project. See AUTHORS file for Copyright information
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
#

set(EXECUTABLE_NAME cmangos_service_realmd)

set(EXECUTABLE_SRCS
    AuthCodes.h
    AuthSocket.cpp
    AuthSocket.h
    Main.cpp
    PatchHandler.cpp
    PatchHandler.h
    RealmList.cpp
    RealmList.h
    SessionManager.cpp
    SessionManager.h
   )

# Add resource file to windows build
if(WIN32)
  set(EXECUTABLE_SRCS ${EXECUTABLE_SRCS} realmd.rc)
endif()

include_directories(
  ${ACE_INCLUDE_DIR}
  ${MYSQL_INCLUDE_DIR}
  ${OPENSSL_INCLUDE_DIR}
  ${CMAKE_SOURCE_DIR}/dep/backtrace
  ${CMAKE_SOURCE_DIR}/src/framework
  ${CMAKE_SOURCE_DIR}/src/framework/configuration
  ${CMAKE_SOURCE_DIR}/src/framework/cryptography
  ${CMAKE_SOURCE_DIR}/src/framework/database
  ${CMAKE_SOURCE_DIR}/src/framework/debugging
  ${CMAKE_SOURCE_DIR}/src/framework/dynamic
  ${CMAKE_SOURCE_DIR}/src/framework/game_system
  ${CMAKE_SOURCE_DIR}/src/framework/log
  ${CMAKE_SOURCE_DIR}/src/framework/network
  ${CMAKE_SOURCE_DIR}/src/framework/platform
  ${CMAKE_SOURCE_DIR}/src/framework/policies
  ${CMAKE_SOURCE_DIR}/src/framework/system
  ${CMAKE_SOURCE_DIR}/src/framework/threading
  ${CMAKE_SOURCE_DIR}/src/framework/utilities
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_BINARY_DIR}
)

add_executable(${EXECUTABLE_NAME} ${EXECUTABLE_SRCS})
set_target_properties(${EXECUTABLE_NAME} PROPERTIES FOLDER Services)

add_dependencies(${EXECUTABLE_NAME} revision.h)
if(WIN32)
  add_dependencies(${EXECUTABLE_NAME} cmangos_dependency_ace)
endif()

target_link_libraries(${EXECUTABLE_NAME}
  ${ACE_LIBRARIES}
  ${Boost_LIBRARIES}
  ${OPENSSL_LIBRARIES}
  cmangos_dependency_backtrace
  cmangos_library_framework
)

if(WIN32)
  target_link_libraries(${EXECUTABLE_NAME}
    optimized ${MYSQL_LIBRARY}
    debug ${MYSQL_DEBUG_LIBRARY}
  )
  if(PLATFORM MATCHES X86)
    target_link_libraries(${EXECUTABLE_NAME})
  endif()
endif()

if(UNIX)
  target_link_libraries(${EXECUTABLE_NAME} ${MYSQL_LIBRARY})
endif()

set(EXECUTABLE_LINK_FLAGS "")

if(UNIX)
  set(EXECUTABLE_LINK_FLAGS "-pthread ${EXECUTABLE_LINK_FLAGS}")
endif()

if(APPLE)
  set(EXECUTABLE_LINK_FLAGS "-framework Carbon ${EXECUTABLE_LINK_FLAGS}")
endif()

set_target_properties(${EXECUTABLE_NAME} PROPERTIES LINK_FLAGS
  "${EXECUTABLE_LINK_FLAGS}"
)

install(TARGETS ${EXECUTABLE_NAME} DESTINATION ${BIN_DIR})
install(FILES realmd.conf.dist.in DESTINATION ${CONF_DIR} RENAME realmd.conf.dist)

if(WIN32 AND MSVC)
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/\${BUILD_TYPE}/${EXECUTABLE_NAME}.pdb DESTINATION ${BIN_DIR} CONFIGURATIONS Debug)
endif()
