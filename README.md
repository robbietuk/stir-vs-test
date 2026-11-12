# STIR from a hand-written Visual Studio solution

Checks whether an installed [STIR](https://github.com/UCL/STIR) can be used from a plain
`.sln` / `.vcxproj` without CMake, as attempted in
[UCL/STIR#1483](https://github.com/UCL/STIR/issues/1483).

The GitHub Actions workflow, on `windows-2022` and `windows-2025`:

1. builds and installs STIR with MSVC (Boost + OpenMP, all optional dependencies off);
2. builds `StirConsumer.sln` with `msbuild`. It compiles STIR's
   `examples/C++/using_installed_STIR/demo_create_image.cxx`;
3. runs the program and checks that it writes `test.hv`.

## What a Visual Studio project needs to use STIR

These are the settings that `find_package(STIR)` adds automatically. See
`consumer/StirConsumer.vcxproj`:

| Setting | Why |
|---|---|
| C++ Language Standard: **C++17** | STIR requires it; VS defaults to C++14, which gives error C2512 in `Array.inl` |
| OpenMP Support: **Yes** | STIR is built with OpenMP by default |
| Runtime library matching STIR's build (`/MD` for Release) | avoids link errors |
| Include dirs: `<install>\include\STIR-x.y` and Boost | headers, including the generated `stir/config.h` |
| Link every `<install>\lib\*.lib` | STIR libraries |
| Compile `<install>\share\STIR-x.y\src\*_registries.cxx` into the program | the registries (file formats, projectors, …) are not in any `.lib`; without them the program fails at runtime |
| Define `CUVEC_DISABLE_CUDA` (non-CUDA builds) | exported by STIR's `stir_registries` target |
