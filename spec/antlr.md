# Source Code

-gen      : generated parser and lexer

-cmake : cmake configure files



# cmake Project

## 用法

src/CMakeLists.txt: 基于antlr生成的C++形式的Parser, Lexer编译

src/cmake: 基于g4文件开始编译。使用时，先将src/cmake/CMakeLists.txt复制并替换src/CMakeLists.txt，进行编译。

## Documentation for FindANTLR

The module defines the following variables:

```
ANTLR_FOUND - true is ANTLR jar executable is found
ANTLR_EXECUTABLE - the path to the ANTLR jar executable
ANTLR_VERSION - the version of ANTLR
```

If ANTLR is found, the module will  provide the macros:

```
ANTLR_TARGET(<name> <input>
             [PACKAGE namespace]
             [OUTPUT_DIRECTORY dir]
             [DEPENDS_ANTLR <target>]
             [COMPILE_FLAGS [args...]]
             [DEPENDS [depends...]]
             [LEXER]
             [PARSER]
             [LISTENER]
             [VISITOR])
```

which creates a custom command to generate C++ files from `<input>`. Running the macro defines the following variables:

```
ANTLR_${name}_INPUT - the ANTLR input used for the macro
ANTLR_${name}_OUTPUTS - the outputs generated by ANTLR
ANTLR_${name}_CXX_OUTPUTS - the C++ outputs generated by ANTLR
ANTLR_${name}_OUTPUT_DIR - the output directory for ANTLR
```

The options are:

* `PACKAGE` - defines a namespace for the generated C++ files
* `OUTPUT_DIRECTORY` - the output directory for the generated files. By default it uses `${CMAKE_CURRENT_BINARY_DIR}`
* `DEPENDS_ANTLR` - the dependent target generated from antlr_target for the current call
* `COMPILE_FLAGS` - additional compile flags for ANTLR tool
* `DEPENDS` - specify the files on which the command depends. It works the same way `DEPENDS` in [`add_custom_command()`](https://cmake.org/cmake/help/v3.11/command/add_custom_command.html)
* `LEXER` - specify that the input file is a lexer grammar
* `PARSER` - specify that the input file is a parser grammar
* `LISTENER` - tell ANTLR tool to generate a parse tree listener
* `VISITOR` - tell ANTLR tool to generate a parse tree visitor

### Examples

To generate C++ files from an ANTLR input file T.g4, which defines both lexer and parser grammar one may call:

```cmake
find_package(ANTLR REQUIRED)
antlr_target(Sample DL.g4)
```

Note that this command will do nothing unless the outputs of `Sample`, i.e. `ANTLR_Sample_CXX_OUTPUTS` gets used by some target.

## Documentation for ExternalAntlr4Cpp

Including ExternalAntlr4Cpp will add `antlr4_static` and `antlr4_shared` as an optional target. It will also define the following variables:

```
ANTLR4_INCLUDE_DIRS - the include directory that should be included when compiling C++ source file
ANTLR4_STATIC_LIBRARIES - path to antlr4 static library
ANTLR4_SHARED_LIBRARIES - path to antlr4 shared library
ANTLR4_RUNTIME_LIBRARIES - path to antlr4 shared runtime library (such as DLL, DYLIB and SO file)
ANTLR4_TAG - branch/tag used for building antlr4 library
```

`ANTLR4_TAG` is set to master branch by default to keep antlr4 updated. However, it will be required to rebuild after every `clean` is called. Set `ANTLR4_TAG` to a desired commit hash value to avoid rebuilding after every `clean` and keep the build stable, at the cost of not automatically update to latest commit.

The ANTLR C++ runtime source is downloaded from GitHub by default. However, users may specify `ANTLR4_ZIP_REPOSITORY` to list the zip file from [ANTLR downloads](http://www.antlr.org/download.html) (under *C++ Target*). This variable can list a zip file included in the project directory; this is useful for maintaining a canonical source for each new build.

Visual C++ compiler users may want to additionally define `ANTLR4_WITH_STATIC_CRT` before including the file. Set `ANTLR4_WITH_STATIC_CRT` to true if ANTLR4 C++ runtime library should be compiled with `/MT` flag, otherwise will be compiled with `/MD` flag. This variable has a default value of `OFF`. Changing `ANTLR4_WITH_STATIC_CRT` after building the library may require reinitialization of CMake or `clean` for the library to get rebuilt.

### Examples

To build and link ANTLR4 static library to a target one may call:

```cmake
include(ExternalAntlr4Cpp)
include_directories(${ANTLR4_INCLUDE_DIRS})
add_executable(output main.cpp)
target_link_libraries(output antlr4_static)
```

It may also be a good idea to copy the runtime libraries (DLL, DYLIB or SO file) to the executable for it to run properly after build. i.e. To build and link antlr4 shared library to a target one may call:

```cmake
include(ExternalAntlr4Cpp)
include_directories(${ANTLR4_INCLUDE_DIRS})
add_executable(output main.cpp)
target_link_libraries(output antlr4_shared)
add_custom_command(TARGET output
                   POST_BUILD
                   COMMAND ${CMAKE_COMMAND}
                           -E copy ${ANTLR4_RUNTIME_LIBRARIES} .
                   WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
```


