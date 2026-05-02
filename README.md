# VS Code C++ Project Generator

A simple Bash script for generating ready-to-use C++ projects on Linux.

The generated project is configured for:

- C++23
- Clang
- clangd
- clang-tidy
- clang-format
- CMake
- Ninja
- VS Code
- CodeLLDB
- Optional vcpkg support

The script is intended for Linux development, especially Arch Linux, but it can also work on other distributions if the required tools are installed.

---

## Features

- Generates a clean C++ project structure
- Creates a basic `src/main.cpp`
- Creates a ready-to-use `CMakeLists.txt`
- Configures VS Code for CMake Tools and clangd
- Adds build and debug tasks for VS Code
- Adds `.clang-format`
- Adds `.clang-tidy`
- Adds `.gitignore`
- Supports optional vcpkg manifest mode with `--vcpkg`
- Does not add any default third-party dependencies

---

## Generated project structure

Without vcpkg:

```text
MyProject/
├── .vscode/
│   ├── launch.json
│   ├── settings.json
│   └── tasks.json
├── src/
│   └── main.cpp
├── .clang-format
├── .clang-tidy
├── .gitignore
└── CMakeLists.txt
```

With vcpkg:

```text
MyProject/
├── .vscode/
│   ├── launch.json
│   ├── settings.json
│   └── tasks.json
├── src/
│   └── main.cpp
├── .clang-format
├── .clang-tidy
├── .gitignore
├── CMakeLists.txt
├── CMakePresets.json
└── vcpkg.json
```

---

## Requirements

Install the required packages.

On Arch Linux:

```bash
sudo pacman -S cmake ninja clang lldb
```

You also need VS Code or a compatible build of it.

Recommended VS Code extensions:

- CMake Tools
- clangd
- CodeLLDB

The Microsoft C/C++ extension is not required because IntelliSense is disabled in favor of clangd.

---

## Installation

Clone this repository or copy the script manually.

Make the script executable:

```bash
chmod +x newcpp
```

Move it to a directory from your `PATH`, for example:

```bash
mkdir -p ~/.local/bin
mv newcpp ~/.local/bin/newcpp
```

Make sure `~/.local/bin` is in your `PATH`.

For Bash or Zsh:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

For Fish:

```fish
fish_add_path ~/.local/bin
```

Now you can use the script from anywhere:

```bash
newcpp MyProject
```

---

## Usage

Create a regular C++ project:

```bash
newcpp MyProject
```

Create a C++ project with vcpkg support:

```bash
newcpp MyProject --vcpkg
```

Show help:

```bash
newcpp --help
```

---

## Regular project

A regular project uses CMake directly without vcpkg.

Build manually:

```bash
cmake -S . -B build -G Ninja \
  -DCMAKE_C_COMPILER=clang \
  -DCMAKE_CXX_COMPILER=clang++ \
  -DCMAKE_BUILD_TYPE=Debug \
  -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

cmake --build build
```

Run:

```bash
./build/MyProject
```

In VS Code, the project can be built using the default build task.

---

## vcpkg support

To create a project with vcpkg support, use:

```bash
newcpp MyProject --vcpkg
```

This creates:

```text
CMakePresets.json
vcpkg.json
```

The generated `vcpkg.json` starts empty:

```json
{
    "name": "MyProject",
    "version-string": "0.1.0",
    "dependencies": []
}
```

No libraries are added automatically.

The script writes the absolute path to the vcpkg CMake toolchain file into `CMakePresets.json`, so VS Code does not need to inherit the `VCPKG_ROOT` environment variable.

---

## vcpkg requirements

Before using `--vcpkg`, make sure vcpkg is installed and `VCPKG_ROOT` is set.

Example:

```bash
export VCPKG_ROOT="$HOME/dev/vcpkg"
```

The script expects this file to exist:

```bash
$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake
```

For permanent configuration, add the export to your shell config.

For Bash:

```bash
echo 'export VCPKG_ROOT="$HOME/dev/vcpkg"' >> ~/.bashrc
```

For Zsh:

```bash
echo 'export VCPKG_ROOT="$HOME/dev/vcpkg"' >> ~/.zshrc
```

For Fish:

```fish
set -Ux VCPKG_ROOT $HOME/dev/vcpkg
```

---

## Building a vcpkg project

Configure:

```bash
cmake --preset debug
```

Build:

```bash
cmake --build --preset debug
```

Run:

```bash
./build/debug/MyProject
```

Release build:

```bash
cmake --preset release
cmake --build --preset release
```

Run release build:

```bash
./build/release/MyProject
```

---

## Adding dependencies with vcpkg

A project generated with `--vcpkg` uses vcpkg manifest mode.

Add a dependency:

```bash
vcpkg add port fmt
```

This updates `vcpkg.json`.

Example:

```json
{
    "name": "MyProject",
    "version-string": "0.1.0",
    "dependencies": [
        "fmt"
    ]
}
```

Then update `CMakeLists.txt` manually according to the package instructions.

Example for `fmt`:

```cmake
find_package(fmt CONFIG REQUIRED)

target_link_libraries(MyProject PRIVATE
    fmt::fmt
)
```

Example `src/main.cpp`:

```cpp
#include <fmt/core.h>

int main() {
    fmt::print("Hello from {}!\n", "fmt");
    return 0;
}
```

Reconfigure and rebuild:

```bash
cmake --preset debug
cmake --build --preset debug
```

Run:

```bash
./build/debug/MyProject
```

Expected output:

```text
Hello from fmt!
```

---

## clang-tidy

The generated project includes a `.clang-tidy` configuration with these groups enabled:

- `bugprone-*`
- `clang-analyzer-*`
- `cppcoreguidelines-*`
- `modernize-*`
- `performance-*`
- `readability-*`

Some checks are disabled to make the generated project more comfortable to use:

```yaml
-modernize-use-trailing-return-type
-cppcoreguidelines-avoid-magic-numbers
-readability-magic-numbers
-cppcoreguidelines-pro-bounds-pointer-arithmetic
-cppcoreguidelines-pro-type-reinterpret-cast
```

This means clang-tidy will not suggest changing normal function declarations like this:

```cpp
int main()
```

into trailing return type style:

```cpp
auto main() -> int
```

---

## clang-format

The generated project uses an LLVM-based `.clang-format` configuration with 4-space indentation and a 100-character column limit.

Formatting on save is enabled in VS Code.

---

## VS Code behavior

When the project is opened in VS Code:

- CMake Tools configures the project automatically
- clangd uses the generated compile commands
- CodeLLDB can debug the executable
- The default build task builds the project

For vcpkg projects, VS Code uses `CMakePresets.json`.

For regular projects, VS Code uses direct CMake settings from `.vscode/settings.json`.

---

## Troubleshooting

### `VCPKG_ROOT is not set`

Set the environment variable before running the script:

```bash
export VCPKG_ROOT="$HOME/dev/vcpkg"
```

Then create the project again:

```bash
newcpp MyProject --vcpkg
```

### `Could not find toolchain file`

Check that this file exists:

```bash
ls "$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake"
```

If it does not exist, your `VCPKG_ROOT` points to the wrong directory.

### `CMake was unable to find a build program corresponding to Ninja`

Install Ninja:

```bash
sudo pacman -S ninja
```

Check that it works:

```bash
ninja --version
```

### VS Code still uses old CMake settings

Delete the build directory:

```bash
rm -rf build
```

Then in VS Code run:

```text
CMake: Delete Cache and Reconfigure
```

Or configure manually:

```bash
cmake --preset debug
```

---

## Notes

The script does not generate a `README.md` inside newly created C++ projects.

This is intentional: generated projects stay minimal and contain only build, editor, formatting, linting, and optional dependency-management configuration.
