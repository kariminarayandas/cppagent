set(AGENT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../src)

if(WIN32)
  # Version info
  string(TIMESTAMP AGENT_VERSION_TIME "%Y-%m-%dT%H:%M:%SZ" UTC)
  set(RESOURCE_FILE "${PROJECT_BINARY_DIR}/agent/version.rc")
  configure_file("${AGENT_SOURCE_DIR}/version.rc.in" "${RESOURCE_FILE}")

  set(WINDOWS_LIBRARIES shlwapi)
else()
  set(RESOURCE_FILE "")
endif()

if(UNIX AND NOT APPLE)
  set(LINUX_LIBRARIES pthread)
endif()

file(GLOB AGENT_SRCS *.cpp)

# Allow better viewing and grouping of files in Visual Studio by defining source groups
source_group("Headers Files" FILES ${AGENT_HEADERS})
source_group("Source Files" FILES ${AGENT_SRCS})
source_group("Dlib Source Files" FILES ${DLIB_FIX_SRCS})
source_group("Resource Files" FILES ${RESOURCE_FILE})

add_executable(agent ${AGENT_SRCS} ${DLIB_FIX_SRCS} ${AGENT_HEADERS} ${RESOURCE_FILE})

AddMsvcXPSupport(agent)
AddLibXML2Support(agent)
AddDLibSupport(agent)
AddDateSupport(agent)
AddJsonSupport(agent)

target_include_directories(agent PRIVATE "${AGENT_SOURCE_DIR}" "${PROJECT_BINARY_DIR}/agent")
target_link_libraries(agent PRIVATE agent_lib PRIVATE ${LINUX_LIBRARIES} ${WINDOWS_LIBRARIES})

if (MSVC)
  add_definitions("/DWINVER=${WINVER}" "/D_WIN32_WINNT=${WINVER}" "/DLIBXML_STATIC")
endif()  

install(TARGETS agent RUNTIME DESTINATION bin)
add_dependencies(agent UPDATE_VERSION)

target_compile_features(agent PRIVATE cxx_std_14)
