# DijoScan

[![.NET](https://img.shields.io/badge/.NET-9.0-512BD4?style=flat-square&logo=dotnet)](https://dotnet.microsoft.com/)
[![Platform](https://img.shields.io/badge/Platform-Windows%20x64-0078D6?style=flat-square&logo=windows)](https://www.microsoft.com/windows)

A modern memory scanner and game modding tool for Windows. Built with .NET 9 and Windows Forms as an alternative to Cheat Engine, designed for game modding, reverse engineering, and security research.

---

## Screenshots

> *Screenshots coming soon*
---

## Features

### Memory Scanning
- **Parallel multi-threaded scanning** for maximum performance
- **Disk-backed storage** supporting 50M+ results without memory exhaustion
- **Value types**: Int8, Int16, Int32, Int64, Float, Double, String (ASCII/Unicode), Array of Bytes
- **Scan types**: Exact, Bigger/Smaller, Between, Unknown, Increased/Decreased/Changed/Unchanged
- **Value freezing** with dedicated high-priority thread for reliable freeze operations

### Memory Browser
- **Hex editor** with live editing powered by HexBox
- **x64 disassembler** using the Iced library
- **Memory region viewer** for process memory layout inspection
- **Pattern search** with wildcard support
- **Memory export** in multiple formats: binary, hex dump, C array

### C# Scripting
- **Roslyn-powered** scripting engine with IntelliSense via RoslynPad
- **Full memory read/write API** for direct process manipulation
- **AOB scanning** with wildcard pattern support
- **Patch helpers**: `Nop()`, `WriteJmp()`, `WriteCall()`, `CreateCodeCave()`
- **Enable/Disable script sections** for toggling patches
- **Save/restore original bytes** for clean deactivation

### Debugging
- **Hardware breakpoints** using debug registers (DR0-DR3)
- **Multiple backends**: Windows Debug API, VEH injection, Kernel driver
- **Register modification** in callbacks, including XMM registers
- **Find what reads/writes** to track memory access

### Managed Runtime Support
| Runtime | Capabilities |
|---------|-------------|
| **Mono/Unity** | Type enumeration, method address resolution via MonoDataCollector |
| **IL2CPP** | Metadata parsing (v24-31), runtime address resolution |
| **.NET Core/5+** | ClrMD-based introspection, heap walking |

### Structure Dissector
- **ReClass.NET-style** memory structure visualization
- **Import types** directly from Mono, IL2CPP, or .NET runtime
- **Code generation**: C#, C++, C headers
- **Live value display** for real-time structure analysis

### Pointer Scanner
- **BFS-based** pointer path finding algorithm
- **Multi-threaded** scanning for improved performance
- **Rescan capability** with new base addresses

### Trainer Generator
- **Export scripts** as standalone executables
- **Publish modes**: Self-contained, Framework-dependent, Native AOT
- **Pre-compiled scripts** with no Roslyn runtime dependency

---

## Requirements

| Requirement | Version |
|-------------|---------|
| Operating System | Windows 10/11 (x64) |
| Runtime | .NET 9.0 |
| Privileges | Administrator |

---

## Installation

1. Download the latest release from the [Releases](../../releases) page
2. Extract the ZIP to a folder of your choice
3. Run `DijoScan.exe` as Administrator

---

## Documentation

See [docs.md](docs.md) for usage instructions, scripting API reference, and examples.

---

## Credits & Acknowledgments

DijoScan is built upon the work of several excellent open-source projects:

- **[Iced](https://github.com/icedland/iced)** - High-performance x86/x64 disassembler
- **[HexBox](https://github.com/nickchal/hexbox)** - Hex editor control for Windows Forms
- **[RoslynPad](https://github.com/roslynpad/roslynpad)** - C# editor with Roslyn-powered IntelliSense
- **[ClrMD](https://github.com/microsoft/clrmd)** - .NET runtime diagnostics library
- **[Cheat Engine](https://github.com/cheat-engine/cheat-engine)** - Inspiration and reference implementation

---

## Disclaimer

This software is intended for educational purposes, legitimate game modding, and authorized security research only. Users are responsible for ensuring their use complies with applicable laws and terms of service.
