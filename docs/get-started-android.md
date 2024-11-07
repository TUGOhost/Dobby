# Getting Started

## create native project and update CMakeLists.txt

```cmake
if(NOT TARGET dobby)
    # 使用相对路径，因为 Dobby 目录就在当前 CMakeLists.txt 同级目录下
    set(DOBBY_DIR ${CMAKE_CURRENT_SOURCE_DIR}/Dobby)

    macro(SET_OPTION option value)
        set(${option} ${value} CACHE INTERNAL "" FORCE)
    endmacro()

    SET_OPTION(DOBBY_DEBUG OFF)
    SET_OPTION(DOBBY_GENERATE_SHARED OFF)

    add_subdirectory(${DOBBY_DIR} dobby)

    get_property(DOBBY_INCLUDE_DIRECTORIES
            TARGET dobby
            PROPERTY INCLUDE_DIRECTORIES)

    include_directories(
            .
            ${DOBBY_INCLUDE_DIRECTORIES}
            $<TARGET_PROPERTY:dobby,INCLUDE_DIRECTORIES>
    )
endif()

add_library(${CMAKE_PROJECT_NAME} SHARED
        native-lib.cpp
        )


target_link_libraries(${CMAKE_PROJECT_NAME}
        dobby
        android
        log)
```

## replace hook function

ref source `builtin-plugin/ApplicationEventMonitor/dynamic_loader_monitor.cc`

ref source `builtin-plugin/ApplicationEventMonitor/posix_file_descriptor_operation_monitor.cc`

## instrument function

ref source `builtin-plugin/ApplicationEventMonitor/memory_operation_instrument.cc`

## Android Linker Restriction

ref source `builtin-plugin/AndroidRestriction/android_restriction_demo.cc`

```c
# impl at SymbolResolver/elf/dobby_symbol_resolver.cc
void *__loader_dlopen = DobbySymbolResolver(NULL, "__loader_dlopen");
DobbyHook((void *)__loader_dlopen, (void *)fake_loader_dlopen, (void **)&orig_loader_dlopen);
```

```
# impl at AndroidRestriction/android_restriction.cc
linker_disable_namespace_restriction();
void *handle = NULL;
handle       = dlopen(lib, RTLD_LAZY);
vm           = dlsym(handle, "_ZN7android14AndroidRuntime7mJavaVME");
```
