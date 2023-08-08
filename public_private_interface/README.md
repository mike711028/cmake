# Public vs Private vs Interface 
在CMake當中的 `target_link_directories` 和 `target_link_libraries` 可以藉由選擇 `PUBLIC` 、 `PRIVATE` 和 `INTERFACE` 來決定header以及library傳遞的屬性

首先敘述一下file structure

```
project
|-- main.cpp
|-- CMakeLists.txt
|-- module_a
|   |-- include 
|   |   |-- func_a.h
|   |
|   |-- src
|   |   |-- func_a.cpp
|   |-- CMakeLists.txt
|   |-- module_b
|       |-- include 
|       |   |-- func_b.h
|       |
|       |-- src
|       |   |-- func_b.cpp
|       |-- CMakeLists.txt
```

以下會以`PRIVATE`、`INTERFACE`以及`PUBLIC`的順序來解釋

## Private 
```
// project/CMakeLists.txt

cmake_minimum_required( VERSION 3.1 )
project( Project )

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/module_a)
add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${PROJECT_NAME} PUBLIC moduleA)

```

```
// project/module_a/CMakeLists.txt

cmake_minimum_required( VERSION 3.1 )
project( ModuleA )

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/module_b)

add_library(${PROJECT_NAME} STATIC ${CMAKE_CURRENT_SOURCE_DIR}/src/func_a.cpp)
target_link_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include/)
target_link_libraries(${PROJECT_NAME} PUBLIC ModuleB)

```

```
// project/module_a/module_b/CMakeLists.txt

cmake_minimum_required( VERSION 3.1 )
project( ModuleB )

add_library(${PROJECT_NAME} STATIC ${CMAKE_CURRENT_SOURCE_DIR}/src/func_b.cpp)
target_link_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include/)

```