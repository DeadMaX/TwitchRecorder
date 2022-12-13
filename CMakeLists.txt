# CMakeList.txt : fichier projet CMake de niveau supérieur, effectuez une configuration globale
# et incluez les sous-projets ici.
#
cmake_minimum_required (VERSION 3.8)

project ("TwitchRecorder")

# Incluez les sous-projets.
# add_subdirectory ("boost")
# add_subdirectory ("cpprestsdk")

set (BOOST_WORK_DIR "${CMAKE_CURRENT_BINARY_DIR}/work-${CMAKE_SYSTEM_PROCESSOR}")
set (BOOST_BASE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/boost")
set (BOOST_LIB_DIR "${BOOST_WORK_DIR}/stage/lib")
set (BOOST_INSTALL_DIR "${BOOST_WORK_DIR}/install")

add_custom_command(OUTPUT "${BOOST_WORK_DIR}/.glue"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${BOOST_WORK_DIR}"
    COMMAND ${CMAKE_COMMAND} -E touch "${BOOST_WORK_DIR}/.glue"
)

add_custom_command(OUTPUT "${BOOST_LIB_DIR}/.glue"
	COMMAND ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=Release -DBOOST_INSTALL_LAYOUT=system -DBOOST_INCLUDE_LIBRARIES="thread\;chrono\;filesystem\;regex\;system\;random" -DCMAKE_INSTALL_PREFIX=${BOOST_INSTALL_DIR} "${BOOST_BASE_DIR}"
    COMMAND ${CMAKE_COMMAND} --build . --config Release
    COMMAND ${CMAKE_COMMAND} -E touch "${BOOST_LIB_DIR}/.glue"
    WORKING_DIRECTORY "${BOOST_WORK_DIR}"
    DEPENDS "${BOOST_WORK_DIR}/.glue"
)

add_custom_command(OUTPUT "${BOOST_INSTALL_DIR}/.glue"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${BOOST_INSTALL_DIR}"   
    COMMAND ${CMAKE_COMMAND} --install . --config Release
    COMMAND ${CMAKE_COMMAND} -E touch "${BOOST_INSTALL_DIR}/.glue"
    WORKING_DIRECTORY "${BOOST_WORK_DIR}"
    DEPENDS "${BOOST_LIB_DIR}/.glue"
)

add_custom_target(build_boost DEPENDS "${BOOST_INSTALL_DIR}/.glue")
add_library(boost INTERFACE)
add_dependencies(boost build_boost)
target_include_directories(boost INTERFACE ${BOOST_INSTALL_DIR}/include)
target_compile_definitions(boost INTERFACE _WIN32_WINNT=0x0A00)
target_link_directories(boost INTERFACE ${BOOST_INSTALL_DIR}/lib)


set (CPPREST_WORK_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpprestwork-${CMAKE_SYSTEM_PROCESSOR}")
set (CPPREST_BASE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/cpprestsdk")
set (CPPREST_LIB_DIR "${CPPREST_WORK_DIR}/Release/Binaries")
set (CPPREST_INSTALL_DIR "${CPPREST_WORK_DIR}/install")

add_custom_command(OUTPUT "${CPPREST_WORK_DIR}/.glue"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${CPPREST_WORK_DIR}"
    COMMAND ${CMAKE_COMMAND} -E touch "${CPPREST_WORK_DIR}/.glue"
)

add_custom_command(OUTPUT "${CPPREST_LIB_DIR}/.glue"
    COMMAND ${CMAKE_COMMAND} -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=${CPPREST_INSTALL_DIR} -DBOOST_ROOT="${BOOST_INSTALL_DIR}" "${CPPREST_BASE_DIR}"
    COMMAND ${CMAKE_COMMAND} --build . --config Release
    COMMAND ${CMAKE_COMMAND} -E touch "${CPPREST_LIB_DIR}/.glue"
    WORKING_DIRECTORY "${CPPREST_WORK_DIR}"
    DEPENDS "${CPPREST_WORK_DIR}/.glue"
)

add_custom_command(OUTPUT "${CPPREST_INSTALL_DIR}/.glue"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${CPPREST_INSTALL_DIR}"   
    COMMAND ${CMAKE_COMMAND} --install . --config Release
    COMMAND ${CMAKE_COMMAND} -E touch "${CPPREST_INSTALL_DIR}/.glue"
    WORKING_DIRECTORY "${CPPREST_WORK_DIR}"
    DEPENDS "${CPPREST_LIB_DIR}/.glue"
)

add_custom_target(build_cpprest DEPENDS "${CPPREST_INSTALL_DIR}/.glue")
add_library(cpprestsdk INTERFACE)
add_dependencies(build_cpprest build_boost)
add_dependencies(cpprestsdk build_cpprest)
target_include_directories(cpprestsdk INTERFACE ${CPPREST_INSTALL_DIR}/include)
target_link_directories(cpprestsdk INTERFACE ${CPPREST_INSTALL_DIR}/lib)
target_link_libraries(cpprestsdk INTERFACE cpprest)

add_subdirectory ("TwitchRecorder")
