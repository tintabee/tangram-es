# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Tangram ES is a C++ library for rendering 2D and 3D maps from vector data using OpenGL ES. It consists of:
- **Core library** (`core/`): Portable C++14 rendering engine with no platform dependencies
- **Platform-specific layers** (`platforms/`): Native implementations for iOS, Android, macOS, Linux, Windows, Raspberry Pi, and Tizen
- **Demo applications**: Sample apps on each platform using the Tangram ES library
- **Test suite**: Unit tests using the Catch framework
- **Benchmarks**: Performance testing framework

The project is designed for modularity: core rendering logic is separated from platform-specific code, allowing easy porting to new platforms.

## Build System

The project uses **CMake** (3.10+) as the primary build system with a **Makefile** wrapper for convenience. The Makefile abstracts platform-specific build commands and works across all supported platforms.

### Platform Targets

Each platform has a dedicated directory under `platforms/` with a `config.cmake` file that:
- Defines platform-specific compiler flags and definitions
- Configures platform-specific source files
- Sets up platform-specific dependencies
- Enables/disables features (e.g., MBTiles datasource, JavaScriptCore)

**Supported platforms:** ios, android, osx (macOS), linux, rpi (Raspberry Pi), windows, tizen

### Build Commands

**Initialize submodules (required before first build):**
```bash
git submodule update --init
```

**Build for current platform (automatically detected):**
```bash
make osx          # macOS command-line app
make linux        # Linux app
make android      # Android demo app
make ios-sim      # iOS simulator
make ios          # iOS device
make rpi          # Raspberry Pi
make xcode        # macOS Xcode project
make ios-xcode    # iOS Xcode workspace
```

**Build configuration:**
- Use `BUILD_TYPE=Debug` or `BUILD_TYPE=Release` (default is Release)
- Use `CMAKE_OPTIONS="-DOPTION=VALUE"` to pass additional CMake options
- Use `make -j` to parallelize builds
- Example: `make linux BUILD_TYPE=Debug CMAKE_OPTIONS="-DTANGRAM_BUILD_TESTS=1"`

**iOS Framework builds:**
```bash
make ios-framework              # Device framework
make ios-framework-sim          # Simulator framework
make ios-xcframework            # Universal XCFramework (device + simulator)
make ios-static-xcframework     # Static library XCFramework
make ios-docs                   # Generate API docs (requires Jazzy)
```

**Android builds:**
```bash
make android-demo               # Demo app debug build
make android-sdk                # SDK release build
make android-install            # Install on connected device
```

### CMake Build Configuration

**Important options:**
- `TANGRAM_PLATFORM=<platform>`: Platform target (auto-detected if not specified)
- `TANGRAM_BUILD_TESTS=1`: Build unit tests
- `TANGRAM_BUILD_BENCHMARKS=1`: Build benchmarks
- `TANGRAM_BUNDLE_TESTS=1` (default ON): Compile all tests into single binary vs. one per test
- `TANGRAM_USE_VCPKG`: Use vcpkg for dependencies (auto-enabled if vcpkg toolchain present)
- `TANGRAM_USE_SYSTEM_FONT_LIBS`: Use system Freetype/ICU/Harfbuzz (Linux only)
- `TANGRAM_USE_SYSTEM_GLFW_LIBS`: Use system GLFW3 (Linux only)
- `TANGRAM_MBTILES_DATASOURCE`: Enable MBTiles data source (default ON)
- `TANGRAM_USE_JSCORE`: Use system JavaScriptCore (iOS/macOS only)
- `TANGRAM_DEV_MODE`: Don't omit frame pointer (for profiling)

### Platform-Specific Setup

**macOS/Linux desktop development:**
```bash
make cmake-osx                  # Configure build
cmake --build build/osx         # Compile
./build/osx/bin/tangram         # Run app
open build/osx/bin/tangram.app  # Or open app bundle
```

**Windows:**
CMake generates Visual Studio 2019 project. Uses vcpkg (reference version 2023.12.12) for dependencies.

**iOS:**
Requires Xcode 9.0+, CMake 3.2+. Can use system Harfbuzz with CoreText support.

**Android:**
Requires Android Studio 3.0+, NDK, CMake, and LLDB. Gradle-based build system.

**Raspberry Pi:**
Uses CMake cross-compilation with toolchain file.

## Testing

**Run all tests:**
```bash
make tests                      # Build tests
./build/tests/tests.out         # Run bundled test suite
```

**Build tests with CMake directly:**
```bash
cmake -H. -Bbuild/tests -DTANGRAM_BUILD_TESTS=1 -DCMAKE_BUILD_TYPE=Debug
cmake --build build/tests
./build/tests/tests.out
```

**Test framework:** Catch2 (header-only, included in `tests/catch/`)

**Test structure:**
- `tests/unit/*.cpp`: Individual unit tests
- `tests/src/mockPlatform.cpp`: Platform mock for testing
- `tests/src/gl_mock.cpp`: OpenGL mock for testing

**Bundled test option:**
By default, all tests compile into a single binary (`tests.out`). Set `TANGRAM_BUNDLE_TESTS=0` to create one executable per test file.

**Test coverage includes:**
- Curl/network handling
- Draw rules and styling
- Duktape JavaScript engine
- File operations
- Camera animations (flyTo)
- Job queue threading
- Labels and label collision
- Layers and scene loading
- Line wrapping
- Mesh and texture operations
- Network data sources
- Scene imports and updates
- Style parameters and mixing
- Tile management
- URL parsing
- YAML filtering and utilities

## Benchmarks

**Build and run benchmarks:**
```bash
make benchmark                  # Build benchmarks
./build/bench/benchmark         # Run with Google Benchmark framework
```

Benchmarks are compiled in Release mode. Use `CMAKE_OPTIONS` to pass custom benchmark arguments.

## Code Style

**Linting and formatting:**
- Use **clang-format** with the provided `.clang-format` config file
- Install: `brew install clang-format` (macOS) or `apt-get install clang-format` (Linux)

**Format changed files:**
```bash
clang-format -i -style=file [file.cpp]
```

**Automated format for git diff:**
```bash
make format                     # Formats all changed files from git diff
```

**Code style guide:**
- C++14 standard
- 4-space indentation
- Column limit: 120 characters
- Follow surrounding code style
- Access modifiers: 4 spaces left of "class"
- Pointer alignment: left
- Brace style: attached
- See `.clang-format` for complete style definition

## Dependency Management

**vcpkg integration:**
- `vcpkg.json` defines all project dependencies
- Dependencies: curl (with SSL/HTTP2), zlib, yaml-cpp, glad, glfw3, benchmark, harfbuzz (with ICU)
- vcpkg is the primary dependency manager for cross-platform builds
- Can optionally use system libraries on Linux with appropriate CMake flags

**Git submodules** (core dependencies, always included):
- `isect2d`: Geometric intersection library
- `css-color-parser-cpp`: CSS color parsing
- `variant`: Header-only variant type
- `earcut`: Polygon triangulation
- `geojson-vt-cpp`: GeoJSON vector tiles
- `duktape`: JavaScript engine
- `yaml-cpp`: YAML parsing
- `alfons`: Text rendering system
- `harfbuzz-icu-freetype`: Font handling (bundled build)
- `SQLiteCpp`: SQLite wrapper for MBTiles
- `glfw`: Window management (desktop platforms)
- `benchmark`: Google Benchmark framework

## Core Architecture

### Layered Design

1. **Platform abstraction layer** (`platforms/common/`):
   - `platform_gl.cpp`: OpenGL ES wrapper
   - Provides common functionality for all platforms

2. **Platform implementations** (`platforms/{platform}/`):
   - Platform-specific window management, input handling, native API bindings
   - Each platform can choose different renderer (OpenGL ES 2/3)

3. **Core library** (`core/`):
   - Fully platform-independent C++14 code
   - No direct system calls or platform dependencies
   - Interfaces via `Platform` class for platform-specific operations

4. **Public API** (`core/include/tangram/`):
   - `map.h`: Main `Map` class for application integration
   - `platform.h`: Platform interface for integration
   - `tangram.h`: Version information and constants
   - `data/`: Data structures (TileSource, Properties, etc.)
   - `util/`: Utility types (LngLat, URL, etc.)

### Key Components

**Rendering pipeline:**
- `src/gl/`: OpenGL ES abstraction (shaders, textures, VAOs, framebuffers)
- `src/style/`: Style system (point, polyline, polygon, text, raster styles)
- `src/scene/`: Scene definition, layers, and styling rules
- `src/view/`: Camera management and transformations

**Data handling:**
- `src/data/`: Data source abstraction (network, cache, raster, GeoJSON, MVT, TopoJSON)
- `src/tile/`: Tile management and processing pipeline
- `src/selection/`: Feature picking and selection queries

**Rendering features:**
- `src/labels/`: Label rendering and collision detection
- `src/marker/`: Marker rendering and management
- `src/text/`: Text rendering and font management
- `src/scene/lights.h`: Lighting system (ambient, directional, point, spot lights)

**Utilities:**
- `src/util/`: Geometry, projections, job queue, YAML parsing, etc.
- `src/debug/`: Debug visualization and frame statistics

### Scene System

Tangram uses YAML-based scene files to define:
- Data sources (tiles, GeoJSON, rasters)
- Layers and feature filtering
- Styling rules (colors, heights, shadows, etc.)
- Lighting and camera defaults

Scene files can be imported and dynamically updated at runtime via `SceneUpdate` mechanism.

### JavaScript Integration

Optional JavaScript engine via Duktape:
- Filter features in scene rules using JavaScript expressions
- `src/js/`: JavaScript binding and integration
- Can be replaced with JavaScriptCore on iOS/macOS

## Development Workflow

### For Desktop (macOS/Linux)

Desktop targets are recommended for core library development:
1. **Setup:** `git submodule update --init` and `brew install cmake` (or apt-get)
2. **Build:** `make osx` or `make linux`
3. **Test:** `make tests && ./build/tests/tests.out`
4. **Format:** `make format` or `clang-format -i -style=file [file]`
5. **Debug:** Use IDE (CLion, Xcode) or GDB/LLDB

### For Platform-Specific Development

- **iOS:** Use Xcode workspace (`make ios-xcode`), edit in Xcode IDE
- **Android:** Use Android Studio, Gradle from command line
- **Windows:** Use CMake with Visual Studio generator

### Code Organization

When making changes:
- **Core library changes:** Modify files in `core/src/` and `core/include/tangram/`
- **Platform integration:** Modify `platforms/{platform}/` and `platforms/common/`
- **Tests:** Add to `tests/unit/` and update `tests/CMakeLists.txt`
- **Scene/styling:** Modify YAML files in `scenes/`

## Continuous Integration

**GitHub Actions workflows** (`.github/workflows/`):
- `pr-checks.yml`: Runs on every push
  - Windows: Visual Studio 2019, vcpkg, CMake
  - macOS: Xcode, Ninja, ccache, tests
  - iOS: Builds simulator and static library variants
- `release.yml`: Builds iOS XCFramework for releases

**CircleCI** (`.circleci/config.yml`):
- Linux: Full build with tests and benchmarks (Docker image: `matteblair/docker-tangram-linux:0.2.0`)
- Android: Gradle builds with ccache
- Runs on push to main branch for release builds

**Build optimization:**
- Uses `ccache` for incremental compilation caching
- Limits concurrent jobs in CI to prevent memory exhaustion
- Windows uses vcpkg NuGet package management

## Release Process

See `release-checklist.md` for complete release workflow. Key steps:
1. Update version numbers in:
   - `platforms/android/tangram/gradle.properties`
   - `platforms/ios/config.cmake`
   - `Tangram-es.podspec`
   - `core/include/tangram.h`
2. Create git tag and push
3. Build artifacts on CI, attach to GitHub release
4. For iOS: push podspec to CocoaPods trunk
5. For Android: release from Sonatype Nexus to Maven Central

