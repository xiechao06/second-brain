---
tags: vcpkg cmake vscode
---
# Vcpkg+CMake+Vscode Setup in Macos

There are 2 ways to setup Vcpkg.

## Globally

### Install vcpkg

```bash
cd
git clone https://github.com/Microsoft/vcpkg.git
~/vcpkg/bootstrap-vcpkg.sh
~/vcpkg/vcpkg integrate install 
```

### Setup environment variables

Edit ~/.bash_profile, add

```bash
export VCPKG_ROOT="$HOME/vcpkg"
export PATH="$PATH:$VCPKG_ROOT"
```

Now, you can execute `vcpkg install` everywhere, and execute `vcpkg install <pkg>` will install the package into `$VCPKG_ROOT/packages/`.

### Execute cmake

You should define `CMAKE_TOOLCHAIN_FILE` and `CMAKE_PREFIX_PATH`[^1] to run `cmake`.

```bash
cmake -DCMAKE_TOOLCHAIN_FILE="$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake" -DCMAKE_PREFIX_PATH="$VCPKG_ROOT/packages" -B build -S
```

Or you could just put CMAKE_TOOLCHAIN_FILE and CMAKE_PREFIX_PATH in ~/.bash_profile

### Setup vscode

You need to tell vscode how to `intellisence` and build.

1\. intellisence & lint

That means you must tell vscode how to find a header files. It is done already by `vcpkg integrate install`. After integration, *vscode cpptools* add *vcpkg* headers automatically in "./c_cpp_properties.json". like:

![vcpkge header](https://user-images.githubusercontent.com/2888536/221072509-dde77806-aea9-43dd-8acc-ae1fd982b9a5.png)

So vscode could find all the headers installed by *vcpkg*.

2\. build

If you have already defined the environment variables `CMAKE_TOOLCHAIN_FILE` and `CMAKE_PREFIX_PATH`, just skip, or you could add setting in vscode:

```json
{
  "cmake.configureSettings": {
    "CMAKE_TOOLCHAIN_FILE": "${userHome}/vcpkg/scripts/buildsystems/vcpkg.cmake"
  }
}
```

## Local in project

If you want let the project manage itself, which is better, you could install `vcpkg` in project.

### Install locally

```bash
git submodule add -b master https://github.com/Microsoft/vcpkg.git
./vcpkg/vcpkg/bootstrap-vcpkg.sh
```

### Setup Vscode

There're 3 ways to setup `CMAKE_TOOLCHAIN_FILE` for vscode.

1\. add following commands in `CMakeLists.txt`

```cmake
set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/vcpkg/scripts/buildsystems/vcpkg.cmake")
```

2\. define `CMAKE_TOOLCHAIN_FILE` in `CMakePresets.json`[^2], like:

```json
"cacheVariables": {
  "CMAKE_TOOLCHAIN_FILE": "${sourceDir}/vcpkg/scripts/buildsystems/vcpkg.cmake",
}
```

3\. define `CMAKE_TOOLCHAIN_FILE` in workspace settings.

```json
{
  "cmake.configureSettings": {
    "CMAKE_TOOLCHAIN_FILE": "${sourceDir}/vcpkg/scripts/buildsystems/vcpkg.cmake"
  }
}
```

I prefer the first way, since it is explicit.

## Resources

* [Cross-Platform Pitfalls and How to Avoid Them](https://www.youtube.com/watch?v=-NhaPNq16Qk&t=1s)
* [Get started with vcpkg](https://vcpkg.io/en/getting-started.html)
* [Configure and build with CMake Presets in Visual Studio Code](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/cmake-presets.md)

## NOTES

* don't use extension *code runner* to run current source file any more, it's not worth setting `-I` and `-L` flags.

[^1]: CMAKE_PREFIX_PATH is needed only for the first time, and I don't know why.

[^2]: You should add `CMakePresets.json` and restart vscode to enable the usage of presets, since `cmake.useCMakePresets` is **auto** by default.
