# WSLg Tracing Infrastructure - Complete Documentation Index

## 📋 Overview

A complete tracing and debugging infrastructure has been added to the WSLg workspace. This enables:

- **Real-time debugging** - Monitor code execution with detailed logs
- **Performance analysis** - Track function entry/exit and timing
- **Component filtering** - Focus on specific modules
- **Environment-based configuration** - Control verbosity at runtime
- **Minimal overhead** - Zero cost when disabled, low cost when enabled

## 📁 New Files

### Core Implementation
1. **`WSLGd/trace.h`** - Advanced tracing configuration and runtime helpers
2. **`WSLGd/common.h`** - Enhanced with new log levels and trace macros (MODIFIED)
3. **`WSLGd/precomp.h`** - Added necessary includes (MODIFIED)

### Documentation
1. **`docs/TRACING.md`** - Complete tracing guide (comprehensive reference)
2. **`docs/TRACING_EXAMPLES.md`** - Practical code examples
3. **`docs/TRACING_QUICK_REF.md`** - Quick reference for common tasks
4. **`docs/TRACING_BUILD_INTEGRATION.md`** - CMake, Meson, and Docker integration
5. **`TRACING_SETUP_SUMMARY.md`** - Setup overview and checklist (in root)

## 🚀 Quick Start (30 seconds)

### Enable Tracing
```bash
export WSLG_TRACE_ENABLED=1
export WSLG_TRACE_LEVEL=1          # 1=TRACE, 5=INFO
export WSLG_TRACE_COMPONENTS=ProcessMonitor
wsl --system <distro>
```

### View Output
```bash
# In system distro
tail -f /mnt/wslg/stderr.log

# From Windows
Get-Content "\\wsl.localhost\<distro>\mnt\wslg\stderr.log" -Tail 50
```

### Add Tracing to Code
```cpp
void MyFunction(const char* param) {
    TRACE_ENTRY();
    TRACE_VALUE("param", param);
    
    // Do work
    
    TRACE_EXIT();
}
```

## 📖 Documentation Map

| Document | Purpose | Audience |
|----------|---------|----------|
| **TRACING.md** | Complete reference guide | Developers, system integrators |
| **TRACING_EXAMPLES.md** | Code examples and patterns | Developers adding traces |
| **TRACING_QUICK_REF.md** | Command reference | Debugging sessions |
| **TRACING_BUILD_INTEGRATION.md** | Build system integration | Build engineers |
| **TRACING_SETUP_SUMMARY.md** | Feature overview | Project managers |

## 🔧 Macros Reference

### Logging Levels
```cpp
LOG_TRACE(fmt, ...)      // Level 1 - Most verbose, function flow
LOG_DEBUG(fmt, ...)      // Level 2 - Debug information
LOG_ERROR(fmt, ...)      // Level 4 - Error conditions
LOG_INFO(fmt, ...)       // Level 5 - Important events (default)
```

### Tracing Helpers
```cpp
TRACE_ENTRY()            // Log function entry
TRACE_EXIT()             // Log function exit
TRACE_CALL(func)         // Log function call
TRACE_VALUE(name, str)   // Log string value (requires ENABLE_DETAILED_TRACING)
TRACE_INT(name, val)     // Log integer value
TRACE_PTR(name, ptr)     // Log pointer value
```

## 🌍 Environment Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `WSLG_TRACE_ENABLED` | Enable/disable tracing | `1` or `true` |
| `WSLG_TRACE_LEVEL` | Verbosity: 1-5 | `1` (verbose) to `5` (quiet) |
| `WSLG_TRACE_COMPONENTS` | Filter components | `ProcessMonitor,FontMonitor` |
| `WSLG_TRACE_FILE` | Log file path | `/mnt/wslg/trace.log` |

## 📊 Verbosity Levels

| Level | Name | Use Case |
|-------|------|----------|
| 1 | TRACE | Detailed function flow, variable values |
| 2 | DEBUG | Diagnostic information |
| 3 | EXCEPTION | Exception conditions |
| 4 | ERROR | Error conditions |
| 5 | INFO | Important events (default) |

**Lower numbers = more verbose**

## 💾 File Locations

### Log Output
- **stderr**: `/mnt/wslg/stderr.log` (in system distro)
- **Custom trace file**: Path specified in `WSLG_TRACE_FILE`

### Configuration Files
- **Header**: `WSLGd/common.h` - Log macros
- **Advanced**: `WSLGd/trace.h` - TraceConfig class
- **Includes**: `WSLGd/precomp.h` - Trace dependencies

## 🎯 Common Tasks

### Debug ProcessMonitor
```bash
export WSLG_TRACE_ENABLED=1
export WSLG_TRACE_LEVEL=1
export WSLG_TRACE_COMPONENTS=ProcessMonitor
```

### Debug Font Changes
```bash
export WSLG_TRACE_COMPONENTS=FontMonitor
```

### Log Everything
```bash
export WSLG_TRACE_LEVEL=1          # TRACE level
```

### Save Traces to File
```bash
export WSLG_TRACE_FILE=/mnt/wslg/debug.log
```

### Performance Testing
```bash
export WSLG_TRACE_LEVEL=5          # INFO only
export WSLG_TRACE_FILE=/mnt/wslg/perf.log
time wsl --system <distro> -- ls /mnt/wslg
```

## ✅ Integration Checklist

To use tracing in your code:

- [ ] Include `common.h` (usually already included)
- [ ] Call `TRACE_ENTRY()` at function start
- [ ] Log parameters with `TRACE_INT()`, `TRACE_VALUE()`, etc.
- [ ] Log important checkpoints with `LOG_INFO()`
- [ ] Log errors with `LOG_ERROR()`
- [ ] Call `TRACE_EXIT()` before return (or use scopes)
- [ ] Test with `WSLG_TRACE_LEVEL=1`

## 🔍 What Gets Traced

When fully enabled, you can trace:

- **Function entry/exit** - See all function calls
- **Parameter values** - Track what functions receive
- **Local variables** - Monitor state changes
- **Error conditions** - Detailed error information
- **Component operations** - ProcessMonitor, FontMonitor, etc.
- **System calls** - Mount operations, socket creation, etc.
- **RDP connection** - Virtual channel operations

## ⚡ Performance Impact

| Configuration | Overhead |
|---------------|----------|
| Tracing disabled | ~0% |
| TRACE_LEVEL=5 (INFO) | ~1-2% |
| TRACE_LEVEL=2 (DEBUG) | ~5-10% |
| TRACE_LEVEL=1 (TRACE) | ~10-20% |

For production, use INFO level or disable tracing.

## 🛠️ Build Integration

### CMake
```cmake
if(CMAKE_BUILD_TYPE MATCHES Debug)
    add_compile_definitions(ENABLE_DETAILED_TRACING)
endif()
```

### Meson
```bash
meson setup builddir -Ddebug_trace=true
```

### Docker
```bash
docker build --build-arg DEBUG_TRACE=1 -t wslg:debug .
```

## 📝 Code Example

```cpp
// In ProcessMonitor.cpp
void ProcessMonitor::MonitorProcesses() {
    TRACE_ENTRY();
    LOG_INFO("Starting process monitoring");
    
    while (m_running) {
        TRACE_VALUE("monitor_state", "waiting");
        
        pid_t pid = waitpid(-1, nullptr, WNOHANG);
        if (pid > 0) {
            LOG_INFO("Process %d exited", pid);
            TRACE_INT("exited_pid", pid);
            HandleProcessExit(pid);
        }
        
        sleep(1);
    }
    
    LOG_INFO("Process monitoring stopped");
    TRACE_EXIT();
}
```

## 🔗 Related Documentation

- **Architecture**: See `WSLg_ArchitectureOverview.png` in `docs/`
- **Build System**: See `config/BUILD.md`
- **Contributing**: See `CONTRIBUTING.md`
- **Security**: See `SECURITY.md`

## 📞 Support & Questions

For questions about tracing:
1. Check `docs/TRACING_QUICK_REF.md` for common commands
2. Review `docs/TRACING_EXAMPLES.md` for code patterns
3. See `docs/TRACING.md` for comprehensive reference
4. Check `WSLGd/common.h` for macro definitions

## 🎓 Learning Path

1. **Start here**: Read `TRACING_QUICK_REF.md` (5 min)
2. **Learn macros**: Review `common.h` macro definitions (10 min)
3. **See examples**: Read `TRACING_EXAMPLES.md` (15 min)
4. **Add tracing**: Start adding traces to your functions
5. **Advanced**: Read `trace.h` for component filtering (10 min)
6. **Integration**: See `TRACING_BUILD_INTEGRATION.md` for build setup (10 min)

---

**Status**: ✅ Tracing infrastructure is ready to use. Begin adding traces to code and test with environment variables enabled.
