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

## CMakeLists.txt configuration
```cmake
// project/CMakeLists.txt

cmake_minimum_required( VERSION 3.1 )
project( Project )

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/module_a)
add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${PROJECT_NAME} PUBLIC moduleA)

```

```cmake
// project/module_a/CMakeLists.txt

cmake_minimum_required( VERSION 3.1 )
project( ModuleA )

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/module_b)

add_library(${PROJECT_NAME} STATIC ${CMAKE_CURRENT_SOURCE_DIR}/src/func_a.cpp)
target_link_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include/)

# 主要是在這利用PUBLIC PRIVATE INTERFACE去改變屬性
target_link_libraries(${PROJECT_NAME} PUBLIC ModuleB)
#target_link_libraries(${PROJECT_NAME} PRIVATE ModuleB)
#target_link_libraries(${PROJECT_NAME} INTERFACE ModuleB)

```

```cmake
// project/module_a/module_b/CMakeLists.txt

cmake_minimum_required( VERSION 3.1 )
project( ModuleB )

add_library(${PROJECT_NAME} STATIC ${CMAKE_CURRENT_SOURCE_DIR}/src/func_b.cpp)
target_link_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include/)

```

以下會以`PRIVATE`、`INTERFACE`以及`PUBLIC`的順序來解釋

## Private 

在被link為 `PRIVATE` 的 `ModuleB` 只能被所要生成的target( `ModuleA` )使用，在其他任何地方引入 `ModuleA` 都不會看到 `ModuleB` 的存在

```cmake
// project/module_a/CMakeLists.txt

cmake_minimum_required( VERSION 3.1 )
project( ModuleA )

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/module_b)

add_library(${PROJECT_NAME} STATIC ${CMAKE_CURRENT_SOURCE_DIR}/src/func_a.cpp)
target_link_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include/)

#target_link_libraries(${PROJECT_NAME} PRIVATE ModuleB)
```

### C++ code explaination 

#### module_b
```cpp
// fun_b.h
#include <iostream>
void Print_B();
```


```cpp
// fun_b.cpp
#include "fun_b.h"
void Print_B()
{
    std::cout << "print from func_b" << std::endl;
}
```

#### module_a
```cpp
// fun_a.h
#include <iostream>
void Print_A();
```


```cpp
// fun_a.cpp
#include "func_a.h"
#include "func_b.h"
void Print_A()
{
    std::cout << "print from func_a" << std::endl;
    Print_B();
}
```

#### project

```cpp
// main.cpp
#include "func_a"
int main()
{
    std::cout << "Hello, world" << std::endl;
    Print_A();
    return 0;
}
```