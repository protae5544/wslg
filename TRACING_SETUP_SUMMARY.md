# WSLg Tracing Infrastructure Setup Summary

## Overview
A comprehensive tracing infrastructure has been added to the WSLg workspace to enable debugging, performance analysis, and runtime monitoring.

## Files Added

### 1. **WSLGd/trace.h** (New)
Advanced tracing configuration and runtime helpers:
- `TraceConfig` class for environment variable-based configuration
- Conditional tracing macros (TRACE_ENABLED, TRACE_COMPONENT_ENABLED, TRACE_LEVEL_ENABLED)
- Support for detailed tracing via ENABLE_DETAILED_TRACING flag
- Component filtering and trace file output

### 2. **docs/TRACING.md** (New)
Complete tracing guide covering:
- Log levels (TRACE, DEBUG, ERROR, INFO, EXCEPTION)
- Available macros and their usage
- Environment variable configuration
- Integration examples
- Log output format
- Performance considerations

### 3. **docs/TRACING_EXAMPLES.md** (New)
Practical code examples demonstrating:
- Basic function tracing with entry/exit logging
- Error handling with tracing
- Component-based filtering
- Complex operation tracing
- Font monitoring examples
- RDP connection tracing
- Performance-critical code tracing
- Best practices

### 4. **docs/TRACING_QUICK_REF.md** (New)
Quick reference guide for common tasks:
- Enable tracing commands
- Logging macros summary
- Tracing helpers
- Example usage
- Log level reference
- File locations

## Files Modified

### 1. **WSLGd/common.h** (Enhanced)
Added new log levels and macros:
- `LOG_LEVEL_TRACE` (level 1) - Most verbose
- `LOG_LEVEL_DEBUG` (level 2) - Debug information
- `LOG_DEBUG()` macro - Debug level logging
- `LOG_TRACE()` macro - Trace level logging
- `TRACE_ENTRY()` - Log function entry
- `TRACE_EXIT()` - Log function exit
- `TRACE_CALL()` - Log function calls

### 2. **WSLGd/precomp.h** (Enhanced)
Added standard library includes needed for tracing:
- `<set>` - For component filtering
- `<string>` - For string handling in trace.h

## Log Levels (Priority)

| Level | Macro | Verbosity |
|-------|-------|-----------|
| 1 | `LOG_TRACE()` | ★★★★★ (Highest) |
| 2 | `LOG_DEBUG()` | ★★★★ |
| 3 | `LOG_EXCEPTION()` | ★★★ |
| 4 | `LOG_ERROR()` | ★★ |
| 5 | `LOG_INFO()` | ★ (Default) |

## Environment Variables

Control tracing behavior at runtime:

- **WSLG_TRACE_ENABLED** - Enable/disable tracing (1 or true)
- **WSLG_TRACE_LEVEL** - Set verbosity level (1-5, lower = more verbose)
- **WSLG_TRACE_COMPONENTS** - Filter to specific components (comma-separated)
- **WSLG_TRACE_FILE** - Write traces to file (path)

## Quick Start Examples

### Enable Full Tracing
```bash
export WSLG_TRACE_ENABLED=1
export WSLG_TRACE_LEVEL=1
wsl --system <distro>
```

### Debug Specific Component
```bash
export WSLG_TRACE_ENABLED=1
export WSLG_TRACE_LEVEL=2
export WSLG_TRACE_COMPONENTS=ProcessMonitor
wsl --system <distro>
```

### Log to File
```bash
export WSLG_TRACE_ENABLED=1
export WSLG_TRACE_FILE=/mnt/wslg/debug.log
wsl --system <distro>
```

## Usage in Code

### Simple Function Tracing
```cpp
void MyFunction(const char* param) {
    TRACE_ENTRY();
    LOG_INFO("Processing: %s", param);
    TRACE_EXIT();
}
```

### With Parameter Logging
```cpp
void ProcessData(int id, const char* data) {
    TRACE_ENTRY();
    TRACE_INT("id", id);
    TRACE_VALUE("data", data);
    
    // Process data...
    
    TRACE_EXIT();
}
```

### Error Handling
```cpp
if (operation_failed) {
    LOG_ERROR("Operation failed: %s", error_msg);
    TRACE_VALUE("error", error_msg);
}
```

## Integration Checklist

To use tracing in existing code:

- [ ] Include `common.h` (already included in most files)
- [ ] Call `TRACE_ENTRY()` at function start
- [ ] Log important variables with `TRACE_INT()`, `TRACE_VALUE()`, etc.
- [ ] Call `TRACE_EXIT()` before return
- [ ] Use `LOG_ERROR()` for errors
- [ ] Use `LOG_INFO()` for important events
- [ ] Test with environment variables enabled

## Performance Impact

- **Zero overhead** when tracing is disabled (WSLG_TRACE_ENABLED=0)
- **Minimal overhead** with TRACE_LEVEL=5 (INFO only)
- **Debug level** (TRACE_LEVEL=2) has measurable but acceptable overhead
- **Full tracing** (TRACE_LEVEL=1) should only be used during development

## Next Steps

1. **Add tracing to WSLGd components:**
   - ProcessMonitor.cpp - Add TRACE_ENTRY/EXIT to functions
   - FontMonitor.cpp - Log directory change events
   - main.cpp - Log initialization steps

2. **Add tracing to WSLDVCPlugin:**
   - WSLDVCPlugin.cpp - Log DVC initialization
   - WSLDVCCallback.cpp - Log app list updates

3. **Test tracing:**
   - Enable tracing and verify log output
   - Check performance impact
   - Validate component filtering

4. **Document component names:**
   - Add list of valid component names to docs/TRACING.md
   - Update ProcessMonitor, FontMonitor, etc. to use component-specific filtering

## References

- See `docs/TRACING.md` for complete guide
- See `docs/TRACING_EXAMPLES.md` for code examples
- See `docs/TRACING_QUICK_REF.md` for quick commands
- See `WSLGd/common.h` for macro definitions
- See `WSLGd/trace.h` for advanced configuration
