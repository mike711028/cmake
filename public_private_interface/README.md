# Public vs Private vs Interface 
在CMake當中的 `target_link_directories` 和 `target_link_libraries` 可以藉由選擇 `PUBLIC` 、 `PRIVATE` 和 `INTERFACE` 來決定header以及library傳遞的屬性

首先敘述一下file structure

```
project
|-- main.cpp
|-- CMakeLists.txt
|-- moduleA
|   |-- include 
|   |   |-- funcA.h
|   |
|   |-- src
|   |   |-- funcA.cpp
|   |-- CMakeLists.txt
|
|-- moduleA
|   |-- include 
|   |   |-- funcB.h
|   |
|   |-- src
|   |   |-- funcB.cpp
|   |-- CMakeLists.txt
```

以下會以`PRIVATE`、`INTERFACE`以及`PUBLIC`的順序來解釋

## Private 

