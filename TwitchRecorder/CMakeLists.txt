# CMakeList.txt : projet CMake pour TwitchRecorder, incluez la source et définissez
# la logique spécifique au projet ici.
#
cmake_minimum_required (VERSION 3.8)

# Ajoutez une source à l'exécutable de ce projet.
add_executable (TwitchRecorder "TwitchRecorder.cpp" "TwitchRecorder.h")
target_link_libraries(TwitchRecorder PUBLIC cpprestsdk)

# TODO: Ajoutez des tests et installez des cibles si nécessaire.
