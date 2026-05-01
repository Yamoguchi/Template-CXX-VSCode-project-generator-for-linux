# VS Code C++ Project Generator for Linux

A simple Bash-based scaffolder for creating ready-to-use C++ projects for Visual Studio Code on Linux.

It generates a C++ project configured for:

- Clang
- clangd
- clang-tidy
- clang-format
- CMake
- Ninja
- VS Code
- CodeLLDB

The goal is to create a complete C++ project with one terminal command, without manually copying folders, writing CMake files, or creating VS Code configuration files every time.

---

## Features

- Generates a complete C++ project structure.
- Creates `CMakeLists.txt` automatically.
- Creates VS Code configuration files:
  - `.vscode/settings.json`
  - `.vscode/tasks.json`
  - `.vscode/launch.json`
- Uses Clang as the default C and C++ compiler.
- Uses Ninja as the CMake generator.
- Enables `compile_commands.json` generation for clangd.
- Enables clangd background indexing.
- Enables clang-tidy diagnostics through clangd.
- Adds `.clang-format` for automatic formatting.
- Adds `.clang-tidy` for static analysis rules.
- Adds `.gitignore`.
- Adds a basic `README.md` to the generated project.
- Opens the created project in VS Code automatically if the `code` command is available.

---

## Requirements

Before using the script, install the required system packages.

### Arch Linux

```bash
sudo pacman -S clang cmake ninja lldb git
```

Required tools:

- `clang`
- `clang++`
- `clangd`
- `clang-tidy`
- `clang-format`
- `cmake`
- `ninja`
- `lldb`
- `git`
- `code`

On Arch Linux, `clangd`, `clang-tidy`, and `clang-format` are provided by the `clang` package.

---

## VS Code Extensions

Install the recommended VS Code extensions:

```bash
code --install-extension llvm-vs-code-extensions.vscode-clangd
code --install-extension ms-vscode.cmake-tools
code --install-extension vadimcn.vscode-lldb
```

Recommended extensions:

- [clangd](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd)
- [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)
- [CodeLLDB](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb)

If you have the Microsoft C/C++ extension installed, it is recommended to disable its IntelliSense engine when using clangd:

```json
"C_Cpp.intelliSenseEngine": "disabled"
```

The generated project already includes this setting.

---

## Installation

The Bash script is included in the root of this repository and is named:

```text
newcpp
```

To install it as a global command, copy it to your local scripts directory.

Create the local scripts directory if it does not exist:

```bash
mkdir -p ~/.local/bin
```

Copy the script from the repository root:

```bash
cp newcpp ~/.local/bin/newcpp
```

Make it executable:

```bash
chmod +x ~/.local/bin/newcpp
```

Make sure `~/.local/bin` is included in your `PATH`.

Check your current `PATH`:

```bash
echo $PATH
```

If `~/.local/bin` is missing, add it.

### Bash

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Zsh

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Fish

```fish
fish_add_path ~/.local/bin
```

After this, the `newcpp` command should be available globally from any terminal.

Check it with:

```bash
which newcpp
```

---

## Usage

Open a terminal in the directory where you want to create a new C++ project.

Run:

```bash
newcpp MyProject
```

This creates a new folder:

```text
MyProject/
```

Then the script automatically opens the project in VS Code if the `code` command is available.

---

## Generated Project Structure

The generated project looks like this:

```text
MyProject/
├── CMakeLists.txt
├── README.md
├── .clang-format
├── .clang-tidy
├── .gitignore
├── .vscode/
│   ├── settings.json
│   ├── tasks.json
│   └── launch.json
└── src/
    └── main.cpp
```

---

## Generated Files

### `CMakeLists.txt`

The generated `CMakeLists.txt` configures the project to use modern C++:

```cmake
cmake_minimum_required(VERSION 3.20)

project(MyProject LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_executable(MyProject
    src/main.cpp
)

target_compile_options(MyProject PRIVATE
    -Wall
    -Wextra
    -Wpedantic
    -Wconversion
    -Wshadow
)
```

---

### `src/main.cpp`

The default source file contains a simple Hello World program:

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << '\n';
    return 0;
}
```

---

### `.vscode/settings.json`

The generated VS Code settings configure CMake and clangd:

```json
{
    "cmake.generator": "Ninja",

    "cmake.configureSettings": {
        "CMAKE_C_COMPILER": "clang",
        "CMAKE_CXX_COMPILER": "clang++",
        "CMAKE_BUILD_TYPE": "Debug",
        "CMAKE_EXPORT_COMPILE_COMMANDS": "ON"
    },

    "cmake.buildDirectory": "${workspaceFolder}/build",
    "cmake.configureOnOpen": true,

    "clangd.path": "/usr/bin/clangd",

    "clangd.arguments": [
        "--background-index",
        "--clang-tidy",
        "--completion-style=detailed",
        "--header-insertion=iwyu",
        "--pch-storage=memory",
        "--compile-commands-dir=${workspaceFolder}/build"
    ],

    "C_Cpp.intelliSenseEngine": "disabled",

    "editor.formatOnSave": true,

    "[cpp]": {
        "editor.defaultFormatter": "llvm-vs-code-extensions.vscode-clangd"
    },

    "[c]": {
        "editor.defaultFormatter": "llvm-vs-code-extensions.vscode-clangd"
    }
}
```

---

### `.vscode/tasks.json`

The generated build task uses CMake:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build project",
            "type": "shell",
            "command": "cmake",
            "args": [
                "--build",
                "${workspaceFolder}/build"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": [
                "$gcc"
            ]
        }
    ]
}
```

You can run it from VS Code with:

```text
Terminal -> Run Build Task
```

Or with the shortcut:

```text
Ctrl + Shift + B
```

---

### `.vscode/launch.json`

The generated debug configuration uses CodeLLDB:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug C++ with CodeLLDB",
            "program": "${workspaceFolder}/build/${workspaceFolderBasename}",
            "args": [],
            "cwd": "${workspaceFolder}",
            "preLaunchTask": "Build project",
            "terminal": "integrated"
        }
    ]
}
```

Use this configuration from the VS Code Run and Debug panel:

```text
Debug C++ with CodeLLDB
```

---

### `.clang-format`

The generated `.clang-format` file uses LLVM style with a few common adjustments:

```yaml
BasedOnStyle: LLVM
IndentWidth: 4
TabWidth: 4
UseTab: Never
ColumnLimit: 100
BreakBeforeBraces: Attach
AllowShortFunctionsOnASingleLine: Empty
AllowShortIfStatementsOnASingleLine: Never
AllowShortLoopsOnASingleLine: false
PointerAlignment: Left
ReferenceAlignment: Left
SortIncludes: true
```

---

### `.clang-tidy`

The generated `.clang-tidy` enables useful checks for modern C++ development:

```yaml
Checks: >
  bugprone-*,
  clang-analyzer-*,
  cppcoreguidelines-*,
  modernize-*,
  performance-*,
  readability-*,
  -cppcoreguidelines-avoid-magic-numbers,
  -readability-magic-numbers,
  -cppcoreguidelines-pro-bounds-pointer-arithmetic,
  -cppcoreguidelines-pro-type-reinterpret-cast

WarningsAsErrors: ''

HeaderFilterRegex: '.*'

FormatStyle: file
```

---

## Building the Project Manually

You can build the generated project from the terminal.

Go into the project directory:

```bash
cd MyProject
```

Configure the project:

```bash
cmake -S . -B build -G Ninja \
  -DCMAKE_C_COMPILER=clang \
  -DCMAKE_CXX_COMPILER=clang++ \
  -DCMAKE_BUILD_TYPE=Debug \
  -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```

Build it:

```bash
cmake --build build
```

---

## Running the Project

After building, run the executable:

```bash
./build/MyProject
```

Expected output:

```text
Hello, World!
```

---

## Debugging

To debug the project in VS Code:

1. Open the generated project in VS Code.
2. Make sure the CodeLLDB extension is installed.
3. Open the Run and Debug panel.
4. Select:

```text
Debug C++ with CodeLLDB
```

5. Press `F5`.

The project will be built before debugging because the debug configuration uses:

```json
"preLaunchTask": "Build project"
```

---

## clangd and `compile_commands.json`

clangd needs `compile_commands.json` to understand the project correctly.

This file contains the real compiler commands used by CMake, including:

- include paths
- compiler flags
- C++ standard
- defines
- source files

The generated project enables this automatically:

```cmake
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

and also through VS Code CMake settings:

```json
"CMAKE_EXPORT_COMPILE_COMMANDS": "ON"
```

After configuring the project, the file should exist here:

```text
build/compile_commands.json
```

If clangd does not work correctly, check that the file exists:

```bash
ls build/compile_commands.json
```

If it does not exist, configure the project again:

```bash
cmake -S . -B build -G Ninja \
  -DCMAKE_C_COMPILER=clang \
  -DCMAKE_CXX_COMPILER=clang++ \
  -DCMAKE_BUILD_TYPE=Debug \
  -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```

Then restart clangd in VS Code:

```text
Ctrl + Shift + P -> clangd: Restart language server
```

---

## Formatting

The project supports automatic formatting on save through clangd and `.clang-format`.

You can also format manually:

```bash
clang-format -i src/main.cpp
```

To check formatting without modifying the file:

```bash
clang-format --dry-run --Werror src/main.cpp
```

---

## Static Analysis

clang-tidy diagnostics are enabled through clangd.

You can also run clang-tidy manually:

```bash
clang-tidy src/main.cpp -p build
```

The `-p build` option tells clang-tidy where to find `compile_commands.json`.

---

## Notes

This script is designed for Linux.

Unlike Windows, Linux executables usually do not use the `.exe` extension. That is why the generated debug configuration uses:

```json
"program": "${workspaceFolder}/build/${workspaceFolderBasename}"
```

instead of:

```json
"program": "${workspaceFolder}/build/${workspaceFolderBasename}.exe"
```

---

## Uninstall

To remove the command, delete the script:

```bash
rm ~/.local/bin/newcpp
```

If you added `~/.local/bin` to your shell configuration only for this script, you can also remove that line from:

- `~/.bashrc`
- `~/.zshrc`
- `~/.config/fish/config.fish`

depending on your shell.
