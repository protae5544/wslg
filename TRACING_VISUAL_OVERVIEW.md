# WSLg Tracing Infrastructure - Visual Overview

## 📊 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    WSLg Tracing System                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────┐      ┌──────────────────────┐    │
│  │  Code Instrumentation│      │  Runtime Configuration│   │
│  ├──────────────────────┤      ├──────────────────────┤    │
│  │ TRACE_ENTRY()        │      │ WSLG_TRACE_ENABLED   │    │
│  │ TRACE_EXIT()         │      │ WSLG_TRACE_LEVEL     │    │
│  │ LOG_TRACE()          │      │ WSLG_TRACE_FILE      │    │
│  │ LOG_DEBUG()          │      │ WSLG_TRACE_COMPONENTS│   │
│  │ LOG_ERROR()          │      │                      │    │
│  │ LOG_INFO()           │      │                      │    │
│  └──────────────────────┘      └──────────────────────┘    │
│         ↓                              ↓                     │
│  ┌──────────────────────┐      ┌──────────────────────┐    │
│  │  common.h & trace.h  │      │  Environment Vars    │    │
│  │  (Macros & Config)   │      │  (TraceConfig)       │    │
│  └──────────────────────┘      └──────────────────────┘    │
│         ↓                              ↓                     │
│  ┌──────────────────────────────────────────────────────┐  │
│  │          LogPrint() - Central Logging               │  │
│  │  [HH:MM:SS.mmm] <LEVEL> WSLGd: func:line: message  │  │
│  └──────────────────────────────────────────────────────┘  │
│         ↓                                                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Output Destinations                                 │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │  stderr → /mnt/wslg/stderr.log (System Distro)      │  │
│  │  file   → WSLG_TRACE_FILE location (custom file)    │  │
│  │  console→ Real-time output in terminal              │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 🎯 Log Level Hierarchy

```
┌─ LEVEL 1: TRACE ────────────────────────────────────────┐
│  • Function entry/exit (TRACE_ENTRY/EXIT)              │
│  • Detailed variable values (TRACE_VALUE/INT/PTR)      │
│  • Internal state transitions                           │
│  • MOST VERBOSE OUTPUT                                 │
└─────────────────────────────────────────────────────────┘
           ↓
┌─ LEVEL 2: DEBUG ────────────────────────────────────────┐
│  • Diagnostic information                               │
│  • Intermediate calculation results                     │
│  • Component-specific operations                       │
└─────────────────────────────────────────────────────────┘
           ↓
┌─ LEVEL 3: EXCEPTION ────────────────────────────────────┐
│  • Exception conditions                                 │
│  • Unusual situations                                  │
└─────────────────────────────────────────────────────────┘
           ↓
┌─ LEVEL 4: ERROR ───────────────────────────────────────┐
│  • Error conditions                                     │
│  • Failed operations                                   │
└─────────────────────────────────────────────────────────┘
           ↓
┌─ LEVEL 5: INFO (DEFAULT) ──────────────────────────────┐
│  • Important events                                     │
│  • Status changes                                      │
│  • Initialization/shutdown                            │
│  • LEAST VERBOSE OUTPUT                                │
└─────────────────────────────────────────────────────────┘
```

## 📁 File Organization

```
WSLg Repository/
├── WSLGd/                          (Main daemon)
│   ├── common.h           ✏️ MODIFIED
│   │   └── LOG_* macros, TRACE_* helpers
│   ├── trace.h            ✨ NEW
│   │   └── TraceConfig class
│   ├── precomp.h          ✏️ MODIFIED
│   │   └── Added <set>, <string>
│   ├── main.cpp           (Can add traces)
│   ├── ProcessMonitor.cpp (Can add traces)
│   └── FontMonitor.cpp    (Can add traces)
│
├── docs/
│   ├── TRACING_INDEX.md        ✨ NEW (Start here!)
│   ├── TRACING_QUICK_REF.md    ✨ NEW (Quick commands)
│   ├── TRACING.md              ✨ NEW (Complete guide)
│   ├── TRACING_EXAMPLES.md     ✨ NEW (Code examples)
│   └── TRACING_BUILD_INTEGRATION.md ✨ NEW (Build setup)
│
├── WSLDVCPlugin/                 (Windows plugin)
│   └── *.cpp                   (Can add traces)
│
├── TRACING_COMPLETE.md           ✨ NEW (Summary)
├── TRACING_SETUP_SUMMARY.md      ✨ NEW (Setup guide)
├── DELIVERABLES.md               ✨ NEW (Full listing)
└── ... (other files)
```

## 🔄 Trace Flow Example

```
User Code:
┌─────────────────────────────┐
│ void ProcessWindow(int id)  │
│ {                            │
│     TRACE_ENTRY();          │◄────┐
│     TRACE_INT("id", id);    │     │
│     HandleWindow();         │     │
│     TRACE_EXIT();           │◄─┐  │
│ }                            │  │  │
└──────────┬──────────────────┘  │  │
           │                      │  │
           ↓                      │  │
      Macro Expansion:           │  │
   LOG_TRACE(...)  ◄─────────────┘  │
      LogPrint() ◄─────────────────┘
           │
           ↓
    Format & Output:
  [14:23:45.123] <1> WSLGd: ProcessWindow:42: >>> Entering
  [14:23:45.124] <1> WSLGd: ProcessWindow:45: id = 5
  [14:23:45.125] <1> WSLGd: ProcessWindow:50: <<< Exiting
           │
           ↓
    /mnt/wslg/stderr.log
    (or custom WSLG_TRACE_FILE)
```

## 🎛️ Configuration Control Points

```
Environment Variables (Set Before Launch)
    │
    ↓
┌─────────────────────────────────────────┐
│  WSLG_TRACE_ENABLED = 1                │  Enable/disable
│  WSLG_TRACE_LEVEL = 1                  │  Verbosity (1-5)
│  WSLG_TRACE_COMPONENTS = ProcessMonitor│  Component filter
│  WSLG_TRACE_FILE = /path/to/file       │  Output file
└─────────────────────────────────────────┘
    │
    ↓
TraceConfig::GetInstance()
    │
    ├─→ IsEnabled()
    ├─→ GetTraceLevel()
    ├─→ IsComponentEnabled(name)
    └─→ GetTraceFile()
    │
    ↓
LogPrint() ← Decision point for filtering
```

## 📊 Usage Matrix

```
┌──────────────────┬──────────────┬──────────────────────┐
│ Need             │ Setting      │ Example              │
├──────────────────┼──────────────┼──────────────────────┤
│ Debug function   │ LEVEL=1      │ TRACE_ENTRY/EXIT     │
│ Debug error      │ LEVEL=4      │ LOG_ERROR            │
│ Debug component  │ COMPONENTS=X │ +LEVEL=2             │
│ Performance test │ FILE=/path   │ LEVEL=5              │
│ Full trace       │ LEVEL=1      │ All output           │
│ Production       │ ENABLED=0    │ No tracing           │
└──────────────────┴──────────────┴──────────────────────┘
```

## 🚀 Quick Start Flow

```
1. Enable Tracing
   export WSLG_TRACE_ENABLED=1
   export WSLG_TRACE_LEVEL=1
        │
        ↓
2. Launch System
   wsl --system <distro>
        │
        ↓
3. View Output
   tail -f /mnt/wslg/stderr.log
        │
        ↓
4. Analyze Results
   [14:23:45.123] <1> WSLGd: function:42: message
        │
        ↓
5. Adjust as Needed
   Change WSLG_TRACE_LEVEL or WSLG_TRACE_COMPONENTS
```

## 📈 Component Trace Levels

```
ProcessMonitor
├── Monitor()          TRACE_ENTRY/EXIT
├── HandleSignal()     TRACE_ENTRY/EXIT
└── RestartProcess()   LOG_ERROR on failure
        │
        ↓
FontMonitor
├── ScanDirectory()    TRACE_ENTRY/EXIT
├── OnChange()         LOG_INFO on change
└── NotifyFonts()      TRACE_CALL
        │
        ↓
WSLGd (main)
├── Initialize()       LOG_INFO/ERROR
├── LaunchWeston()     TRACE_ENTRY/EXIT
└── LaunchRDP()        LOG_ERROR on failure
```

## 🔍 Search & Filter Guide

```
View all traces:
  tail /mnt/wslg/stderr.log

View TRACE level only:
  grep "<1>" /mnt/wslg/stderr.log

View errors only:
  grep "<4>" /mnt/wslg/stderr.log

View ProcessMonitor:
  grep "ProcessMonitor" /mnt/wslg/stderr.log

View function entry/exit:
  grep ">>> Entering\|<<< Exiting" /mnt/wslg/stderr.log
```

## 📚 Documentation Quick Links

```
START HERE
    │
    ├─→ TRACING_QUICK_REF.md (2 min read)
    │
    ├─→ Need examples?
    │   └─→ TRACING_EXAMPLES.md
    │
    ├─→ Need complete reference?
    │   └─→ TRACING.md
    │
    ├─→ Need build integration?
    │   └─→ TRACING_BUILD_INTEGRATION.md
    │
    └─→ Need overview?
        └─→ TRACING_COMPLETE.md
```

## ✅ Implementation Status

```
DONE ✅
├─ Infrastructure implemented
├─ Macros and helpers added
├─ Environment variable support
├─ Component filtering
├─ Documentation (6 files)
├─ Code examples (15+)
├─ Build integration guide
└─ Quick start guide

TODO (Optional Next Steps)
├─ Add traces to ProcessMonitor.cpp
├─ Add traces to FontMonitor.cpp
├─ Add traces to WSLDVCPlugin
├─ Define component names list
├─ Performance baseline test
└─ Production deployment
```

---

**Ready to use!** Start with `docs/TRACING_QUICK_REF.md` or run:
```bash
export WSLG_TRACE_ENABLED=1 && export WSLG_TRACE_LEVEL=1 && wsl --system <distro>
```
