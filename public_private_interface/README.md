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

被 `target_link_libraries` 為 `PRIVATE` 的 `ModuleB` 只能被所要生成的target( `ModuleA` )使用，在其他任何地方引入 `ModuleA` 都不會看到 `ModuleB` 的存在

### C++ code explaination 

#### module_b
```cpp
// func_b.h
#include <iostream>
void Print_B();
```

```cpp
// func_b.cpp
#include "func_b.h"
void Print_B()
{
    std::cout << "print from func_b" << std::endl;
}
```

#### module_a
把 `ModuleB` 設為 `PRIVATE` 代表只限定於 `ModuleA` 內使用，所以 `ModuleB` 的API**只能**在 `ModuleA` 的實作(.cpp)時引入，如果在定義(.h)
內引入後，當其他地方再引入 `ModuleA` 的標頭檔時就會發生報錯(cmake可以過，但make報錯)

```cpp
// func_a.h
#include <iostream>
void Print_A();
```

```cpp
// func_a.cpp
#include "func_a.h"
#include "func_b.h" // include func_b.h in PRIVATE
void Print_A()
{
    std::cout << "print from func_a" << std::endl;
    Print_B(); // use function of ModuleB only in implementation 
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

#### result

```console
Hello, world
print from func_a
print from func_b
```

## Interface

被 `target_link_libraries` 為 `INTERFACE` 的 `ModuleB` **不能**在target( `ModuleA` )內使用，只有當其他地方引入 `ModuleA` 時才會看到 `ModuleB` 的存在

### C++ code explaination 

#### module_b
```cpp
// func_b.h
#include <iostream>
void Print_B();
```

```cpp
// func_b.cpp
#include "func_b.h"
void Print_B()
{
    std::cout << "print from func_b" << std::endl;
}
```

#### module_a
把 `ModuleB` 設為 `INTERFACE` 代表只是將 `ModuleA` 當作介面，所以 `ModuleB` 的API不能在 `ModuleA` 的實作(.cpp)時引入，**只能**在定義(.h)
內引入後，再由其他地方引入 `ModuleA` 的標頭檔來利用`ModuleB` 的API

```cpp
// func_a.h
#include <iostream>
#include "func_b.h" // use func_a.h as the interface for other target
void Print_A();
```

```cpp
// func_a.cpp
#include <iostream>
// can not include func_a.h because will include func_b.h too
// and func_b.h should not show in the implementation due to INTERFACE
void Print_A()
{
    std::cout << "print from func_a" << std::endl;
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
    Print_B(); // can see and call API of ModuleB through ModuleA
    return 0;
}
```

#### result

```console
Hello, world
print from func_a
print from func_b
```


## Public

被 `target_link_libraries` 為 `PUBLIC` 的 `ModuleB` 可以在target( `ModuleA` )內使用，在當其他地方引入 `ModuleA` 時也能看到 `ModuleB` 的存在

```
PUBLIC = PRIVATE + INTERFACE
```

### C++ code explaination 

#### module_b
```cpp
// func_b.h
#include <iostream>
void Print_B();
```

```cpp
// func_b.cpp
#include "func_b.h"
void Print_B()
{
    std::cout << "print from func_b" << std::endl;
}
```

#### module_a
把 `ModuleB` 設為 `PUBLIC` 代表 `ModuleB` 的API既可以在 `ModuleA` 的實作(.cpp)時引入，也可以在定義(.h)內引入後，再由其他地方引入 `ModuleA` 的標頭檔來利用`ModuleB` 的API

```cpp
// func_a.h
#include <iostream>
#include "func_b.h" // include func_b.h here for implementation and call from other places
void Print_A();
```

```cpp
// func_a.cpp
#include <iostream>
#include "func_a.h"

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
    Print_A(); // also call Print_B() by Print_A()
    Print_B(); // can see and call API of ModuleB through ModuleA
    return 0;
}
```

#### result

```console
Hello, world
print from func_a
print from func_b
print from func_b
```

## Conclusion

* `PRIVATE`: `ModuleB` 只連結到 `ModuleA` ，如果 `Project` 連結了 `ModuleA`，`Project` 不連結也看不見 `ModuleB`


* `INTERFACE`: `ModuleB` 不會連結到 `ModuleA` ，一旦 `Project` 連結了 `ModuleA`，`Project` 就會連結也看得見 `ModuleB`

* `PUBLIC`: `ModuleB` 會連結到 `ModuleA` ， `Project` 連結了 `ModuleA`，`Project` 也會跟著連結 `ModuleB

* `target_include_directories` : 負責控制標頭檔的傳遞
* `target_link_libraries` : 負責控制連結庫和標頭檔的傳遞
