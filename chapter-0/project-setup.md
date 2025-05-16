---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Project Setup

## Project Structure

<pre><code>jp-engine/
├── engine/
│   ├── app.cc
│   └── CMakeLists.txt
├── game/
│   ├── main.cc
│   └── CMakeLists.txt
├── CMakeLists.txt
<strong>├── .clang-format
</strong>├── .clang-tidy
└── .gitignore
</code></pre>

* `engine/`: This directory will house our game engine's core code.
* `game/`: This directory contains the game's executable code, which uses the engine.
* `CMakeLists.txt`: This is the top-level CMake configuration, orchestrating the build process.
* `.clang-format`: Defines the formatting rules.
* `.clang-tidy`: Defines the static analysis rules.
* `.gitignore`: Used by git to determine which files or directories to ignore in the source control.

### CMake Files

{% code title="CMakeLists.txt (Top-Level CMake)" %}
```cmake
# Specifies the minimum CMake version required to build the project.
cmake_minimum_required(VERSION 3.28)

# Sets the name of the project.
project(jp-engine)

# Adds the 'engine' subdirectory to the build process.
add_subdirectory(engine)
# Adds the 'game' subdirectory to the build process.
add_subdirectory(game)

# Finds the 'clang-tidy' executable.
find_program(CLANG_TIDY_EXE NAMES "clang-tidy")
# Checks if 'clang-tidy' was found.
if (CLANG_TIDY_EXE)
    # Sets a variable to the path of the 'clang-tidy' executable.
    set(CLANG_TIDY_COMMAND "${CLANG_TIDY_EXE}")

    # Enables clang-tidy checks for the 'engine' target.
    set_target_properties(engine PROPERTIES CXX_CLANG_TIDY "${CLANG_TIDY_COMMAND}")
    # Enables clang-tidy checks for the 'game' target.
    set_target_properties(game PROPERTIES CXX_CLANG_TIDY "${CLANG_TIDY_COMMAND}")
else()
    # Displays a message if 'clang-tidy' was not found.
    message("Clang Tidy not found")
endif()
```
{% endcode %}

The root CMake script configures the whole build process and adds two sub-directories, `engine` and `game` to the build. After that, if found, it attempts to locate the `clang-tidy` static analysis tool and enables it for both targets.

{% code title="engine/CMakeLists.txt" %}
```cmake
# Include the FetchContent module for managing external dependencies
include(FetchContent)
# Declare the SFML library as a dependency to be fetched
FetchContent_Declare(SFML
    GIT_REPOSITORY https://github.com/SFML/SFML.git
    GIT_TAG 3.0.1
    GIT_SHALLOW ON
    EXCLUDE_FROM_ALL
    SYSTEM)

# Make the declared SFML dependency available for use in the project
FetchContent_MakeAvailable(SFML)

# Create a library named 'engine' from the source file 'app.cc'. More source files will be added
add_library(engine app.cc)

# Specify that the 'engine' target requires at least C++23 standard
target_compile_features(engine PRIVATE cxx_std_23)
# Disable automatic C++ extensions for the 'engine' target
set_target_properties(engine PROPERTIES CXX_EXTENSIONS OFF)
# Set private compile options for the 'engine' target, specific to the compiler
target_compile_options(engine PRIVATE
  # Enable /W4 warning level for MSVC compiler
  $<$<CXX_COMPILER_ID:MSVC>:/W4>
  # Enable -Wall, -Wextra, and -Wpedantic warnings for non-MSVC compilers (Clang, G++)
  $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Wpedantic>
)

# Link the 'engine' library with the SFML Audio and Graphics components
target_link_libraries(engine PUBLIC SFML::Audio SFML::Graphics)
# Add the parent source directory to the public include paths for the 'engine' target
target_include_directories(engine PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/..")
```
{% endcode %}

This CMake script makes available the SFML library via `FetchContent`  and configures a library named`engine`. The target requires C++23 and compiler-specific warning flags (extra warnings and no compiler extensions). The `engine` library is linked against SFML's Audio and Graphics components so that they are available in the code. The parent source directory is added to the include paths so that the C++ includes will look like `#include "engine/some_file.h"`.

{% code title="game/CMakeLists.txt" %}
```cmake
# Create an executable named 'game' from the source file 'main.cc'. More source files will be added.
add_executable(game main.cc)
# Specify that the 'game' target requires at least C++23 standard
target_compile_features(game PRIVATE cxx_std_23)
# Disable automatic C++ extensions for the 'game' target
set_target_properties(game PROPERTIES CXX_EXTENSIONS OFF)

# Set private compile options for the 'game' target, specific to the compiler
target_compile_options(game PRIVATE
  # Enable /W4 warning level
  $<$<CXX_COMPILER_ID:MSVC>:/W4>
  # Enable -Wall, -Wextra, and -Wpedantic warnings for non-MSVC compilers
  $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Wpedantic>
)

# Apply compiler and linker options specifically for non-MSVC compilers
if (NOT MSVC)
  # Enable address and undefined behavior sanitizers in Debug configuration
  target_compile_options(game PRIVATE $<$<CONFIG:DEBUG>:-fsanitize=address,undefined>)
  target_link_options(game PRIVATE $<$<CONFIG:DEBUG>:-fsanitize=address,undefined>)
endif()

# Link the 'game' executable with the 'engine' library
target_link_libraries(game PRIVATE engine)
```
{% endcode %}

This CMake script configures the build process for an executable named `game`. Most of the compiler flags are the same as the `engine` ones, except that in the `game` project, for non-MSVC compilers, in Debug mode, it enables the address and undefined behavior sanitizers for runtime error detection. These are extremely useful for finding memory leaks and undefined behaviour. Finally, the `game` executable is linked against the `engine` library. Later, we'll add one more command to this script to copy the resources directory containing the game assets.&#x20;

Note: As you add more source files to the engine and game projects, <mark style="color:red;">**remember**</mark> to also update the `add_library` and `add_executable` commands in their respective `CMakeLists.txt` files. Failing to do so will prevent these new files from being compiled, leading to **undefined reference** errors during the linking stage.

### Source Files

{% code title="engine/app.cc" %}
```cpp
// Leave empty for now.
```
{% endcode %}

{% code title="game/main.cc" %}
```cpp
int main() {
  return 0;
}
```
{% endcode %}

Pretty much empty source files, for now.

### Configuration Files

If you don't use some of the following tools, you can skip them.

{% code title=".clang-format" %}
```yaml
# From: https://github.com/kehanXue/google-style-clang-format
# Google C/C++ Code Style settings
# https://clang.llvm.org/docs/ClangFormatStyleOptions.html
# Author: Kehan Xue, kehan.xue (at) gmail.com

Language: Cpp
BasedOnStyle: Google
AccessModifierOffset: -1
AlignAfterOpenBracket: Align
AlignConsecutiveAssignments: None
AlignOperands: Align
AllowAllArgumentsOnNextLine: true
AllowAllConstructorInitializersOnNextLine: true
AllowAllParametersOfDeclarationOnNextLine: false
AllowShortBlocksOnASingleLine: Empty
AllowShortCaseLabelsOnASingleLine: false
AllowShortFunctionsOnASingleLine: Inline
AllowShortIfStatementsOnASingleLine: Never # To avoid conflict, set this "Never" and each "if statement" should include brace when coding
AllowShortLambdasOnASingleLine: Inline
AllowShortLoopsOnASingleLine: false
AlwaysBreakAfterReturnType: None
AlwaysBreakTemplateDeclarations: Yes
BinPackArguments: true
BreakBeforeBraces: Custom
BraceWrapping:
  AfterCaseLabel: false
  AfterClass: false
  AfterStruct: false
  AfterControlStatement: Never
  AfterEnum: false
  AfterFunction: false
  AfterNamespace: false
  AfterUnion: false
  AfterExternBlock: false
  BeforeCatch: false
  BeforeElse: false
  BeforeLambdaBody: false
  IndentBraces: false
  SplitEmptyFunction: false
  SplitEmptyRecord: false
  SplitEmptyNamespace: false
BreakBeforeBinaryOperators: None
BreakBeforeTernaryOperators: true
BreakConstructorInitializers: BeforeColon
BreakInheritanceList: BeforeColon
ColumnLimit: 80
CompactNamespaces: false
ContinuationIndentWidth: 4
Cpp11BracedListStyle: true
DerivePointerAlignment: false # Make sure the * or & align on the left
EmptyLineBeforeAccessModifier: LogicalBlock
FixNamespaceComments: true
IncludeBlocks: Preserve
IndentCaseLabels: true
IndentPPDirectives: None
IndentWidth: 2
KeepEmptyLinesAtTheStartOfBlocks: true
MaxEmptyLinesToKeep: 1
NamespaceIndentation: None
ObjCSpaceAfterProperty: false
ObjCSpaceBeforeProtocolList: true
PointerAlignment: Left
ReflowComments: false
SeparateDefinitionBlocks: Always # Only support since clang-format 14
SpaceAfterCStyleCast: false
SpaceAfterLogicalNot: false
SpaceAfterTemplateKeyword: true
SpaceBeforeAssignmentOperators: true
SpaceBeforeCpp11BracedList: false
SpaceBeforeCtorInitializerColon: true
SpaceBeforeInheritanceColon: true
SpaceBeforeParens: ControlStatements
SpaceBeforeRangeBasedForLoopColon: true
SpaceBeforeSquareBrackets: false
SpaceInEmptyParentheses: false
SpacesBeforeTrailingComments: 2
SpacesInAngles: false
SpacesInCStyleCastParentheses: false
SpacesInContainerLiterals: false
SpacesInParentheses: false
SpacesInSquareBrackets: false
Standard: c++11
TabWidth: 4
UseTab: Never
```
{% endcode %}

This YAML file configures the code formatting style for C++ code using `clang-format`. It is based on the Google C++ style guide.

{% code title=".clang-tidy" %}
```yaml
---

# List of checks to enable or disable.
# Checks starting with '-' are disabled.
# '*' enables all default checks.
# Specific checks can be enabled or disabled by name.
Checks: "*,
  -fuchsia-*,
  -altera-*,
  -boost-*,
  -zircon-*,
  -abseil-*,
  -modernize-use-trailing-return-type,
  -cppcoreguidelines-avoid-magic-numbers,
  -readability-magic-numbers,
  -misc-const-correctness,
  -misc-no-recursion,
  -readability-identifier-length,
  -readability-static-definition-in-anonymous-namespace,
  -clang-analyzer-optin.core.EnumCastOutOfRange,
  -llvm-*,
  -llvmlibc-*"

# Regular expression to filter header files that clang-tidy will analyze.
# ".*" means all header files will be considered.
HeaderFilterRegex: ".*"

# Specifies the code formatting style to be used by clang-tidy (if any formatting checks are enabled).
FormatStyle: none
```
{% endcode %}

This YAML file configures the `clang-tidy` static analysis tool.

{% code title=".gitignore" %}
```
build
out
.cache
.idea
.vs
.vscode
.DS_Store
```
{% endcode %}

And this is the usual `.gitignore` file used to avoid git tracking the build directory and the code editor-specific config files.
