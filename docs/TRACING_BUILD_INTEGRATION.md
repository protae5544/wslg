# WSLg Tracing Build Integration

## Compilation Flags

To enable detailed tracing macros at compile time, add the following flag to your build:

```bash
-DENABLE_DETAILED_TRACING
```

This enables:
- `TRACE_FUNC_ENTRY()` - Function entry logging
- `TRACE_FUNC_EXIT()` - Function exit logging
- `TRACE_VALUE(name, str)` - String value logging
- `TRACE_INT(name, val)` - Integer value logging
- `TRACE_PTR(name, ptr)` - Pointer value logging

## CMake Integration

If using CMake for WSLGd builds, add to your CMakeLists.txt:

```cmake
# Optional: Enable detailed tracing (debug builds)
if(CMAKE_BUILD_TYPE MATCHES Debug)
    add_compile_definitions(ENABLE_DETAILED_TRACING)
endif()

# Or always enable:
add_compile_definitions(ENABLE_DETAILED_TRACING)

# Add trace.h to include directories
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/WSLGd)
```

## Meson Integration

If using Meson for vendor builds, add to meson.build:

```meson
# Optional debug tracing
debug_trace = get_option('debug_trace')
if debug_trace
  add_project_arguments('-DENABLE_DETAILED_TRACING', language: 'cpp')
endif()
```

And in meson_options.txt:

```ini
option('debug_trace', type: 'boolean', value: false,
       description: 'Enable detailed function tracing')
```

Build with:
```bash
meson setup builddir -Ddebug_trace=true
```

## Build-time Configuration

### Release Build (Production)
No compile-time flags needed. Runtime configuration via environment variables.

```bash
# WSLGd compilation
gcc -O2 -o wslgd WSLGd/*.cpp WSLGd/common.cpp
```

### Debug Build (Development)
Enable detailed tracing:

```bash
# WSLGd compilation with detailed tracing
gcc -O0 -g -DENABLE_DETAILED_TRACING -o wslgd_debug WSLGd/*.cpp WSLGd/common.cpp
```

### Conditional Compilation Example

In code, you can conditionally compile tracing:

```cpp
#ifdef ENABLE_DETAILED_TRACING
    #define TRACE_FUNC_ENTRY() TRACE_ENTRY()
    #define TRACE_FUNC_EXIT() TRACE_EXIT()
#else
    #define TRACE_FUNC_ENTRY()
    #define TRACE_FUNC_EXIT()
#endif
```

## Header Dependencies

The tracing infrastructure requires:

**In WSLGd/precomp.h:**
- `<set>` - For component name tracking
- `<string>` - For string operations
- `<cstdlib>` - For getenv()
- `<cstring>` - For strcmp()

All are already included in the updated precomp.h.

## No External Dependencies

Tracing uses only standard C++ library functions:
- `std::set` - Component name storage
- `std::string` - String manipulation
- `std::getenv()` - Environment variable access
- Standard C functions (sprintf, va_args, etc.)

No additional libraries or third-party packages required.

## Build Verification

After building, verify tracing is available:

```bash
# Check that WSLGd links correctly
ldd ./wslgd | head -20

# Test tracing at runtime
export WSLG_TRACE_ENABLED=1
export WSLG_TRACE_LEVEL=1
wsl --system <distro>
```

## Docker Build Integration

In the Dockerfile or container build script (config/BUILD.md):

```dockerfile
# Optional: Build with detailed tracing for debugging
ARG DEBUG_TRACE=0
RUN if [ "$DEBUG_TRACE" = "1" ]; then \
      CXXFLAGS="-DENABLE_DETAILED_TRACING" make -j$(nproc) wslgd; \
    else \
      make -j$(nproc) wslgd; \
    fi
```

Build with tracing:
```bash
docker build --build-arg DEBUG_TRACE=1 -t wslg:debug .
```

## Continuous Integration

For CI/CD pipelines (azure-pipelines.yml):

```yaml
- task: CmdLine@2
  displayName: 'Build WSLGd with tracing'
  inputs:
    script: |
      cd WSLGd
      make clean
      make CXXFLAGS="-DENABLE_DETAILED_TRACING" -j4
      
- task: CmdLine@2
  displayName: 'Test tracing'
  inputs:
    script: |
      export WSLG_TRACE_ENABLED=1
      export WSLG_TRACE_LEVEL=2
      ./test_runner.sh
```

## Performance Profiling

Use tracing to profile performance:

```bash
# Measure startup time
export WSLG_TRACE_ENABLED=1
export WSLG_TRACE_LEVEL=1
export WSLG_TRACE_FILE=/mnt/wslg/startup_trace.log

time wsl --system <distro> -- ls /mnt/wslg

# Analyze trace file
grep ">>> Entering\|<<< Exiting" /mnt/wslg/startup_trace.log
```

## Debugging with GDB

When debugging with gdb, tracing helps correlate code execution:

```bash
# Start with detailed tracing
export WSLG_TRACE_ENABLED=1
export WSLG_TRACE_LEVEL=1

# Launch gdb with WSLGd
gdb --args ./wslgd --debug

# Inside gdb
(gdb) break ProcessMonitor.cpp:50
(gdb) run
(gdb) next  # Traces appear in parallel terminal
```

## Memory Profiling

Tracing overhead is minimal:

- `TRACE_ENTRY()` - Single fprintf() call (~100 bytes stack)
- `LOG_TRACE()` - One printf format string (~200 bytes)
- No dynamic memory allocation in tracing code
- Suitable for systems with limited memory

## Recommended Build Configurations

### Development / Debugging
```bash
cmake .. \
  -DCMAKE_BUILD_TYPE=Debug \
  -DENABLE_DETAILED_TRACING=ON \
  -DCMAKE_C_FLAGS="-g -O0" \
  -DCMAKE_CXX_FLAGS="-g -O0"
```

### Testing
```bash
cmake .. \
  -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DENABLE_DETAILED_TRACING=ON \
  -DCMAKE_C_FLAGS="-g -O2" \
  -DCMAKE_CXX_FLAGS="-g -O2"
```

### Production
```bash
cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DENABLE_DETAILED_TRACING=OFF \
  -DCMAKE_C_FLAGS="-O3" \
  -DCMAKE_CXX_FLAGS="-O3"
```
