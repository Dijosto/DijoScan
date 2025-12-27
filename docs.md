# ScriptContext API Documentation

Complete reference for the C# scripting API available in the Script Editor and Trainers.

## Table of Contents

- [Overview](#overview)
- [Basic Properties](#basic-properties)
- [Memory Read Operations](#memory-read-operations)
- [Memory Write Operations](#memory-write-operations)
- [Extended Memory Types](#extended-memory-types)
- [AOB/Pattern Scanning](#aobpattern-scanning)
- [Patch Helpers](#patch-helpers)
- [Save/Restore System](#saverestore-system)
- [Memory Allocation](#memory-allocation)
- [Pointer Operations](#pointer-operations)
- [Module Helpers](#module-helpers)
- [Freeze API](#freeze-api)
- [Breakpoint API](#breakpoint-api)
- [Managed Runtime Support](#managed-runtime-support)
- [IL2CPP Support](#il2cpp-support)
- [Script Sections](#script-sections)
- [Complete Examples](#complete-examples)

---

## Overview

The `ScriptContext` class provides all memory operations and utilities for scripts. It's available as the global context in the Script Editor and as `TrainerScriptContext` in exported trainers.

```csharp
// All methods are called directly - no "context." prefix needed
Print("Hello from script!");
var health = ReadInt32(0x12345678);
```

---

## Basic Properties

| Property | Type | Description |
|----------|------|-------------|
| `ProcessName` | `string` | Name of the attached process |
| `ProcessId` | `int` | Process ID |
| `IsProcessRunning` | `bool` | Whether the process is still running |

```csharp
Print($"Attached to: {ProcessName} (PID: {ProcessId})");
if (!IsProcessRunning) {
    Print("Process has exited!");
}
```

---

## Memory Read Operations

### Basic Types

| Method | Return Type | Description |
|--------|-------------|-------------|
| `ReadByte(address)` | `byte?` | Read 1 byte |
| `ReadBytes(address, count)` | `byte[]?` | Read byte array |
| `ReadInt32(address)` | `int?` | Read 4-byte signed integer |
| `ReadInt64(address)` | `long?` | Read 8-byte signed integer |
| `ReadFloat(address)` | `float?` | Read 4-byte float |
| `ReadDouble(address)` | `double?` | Read 8-byte double |
| `ReadUInt16(address)` | `ushort?` | Read 2-byte unsigned integer |
| `ReadUInt32(address)` | `uint?` | Read 4-byte unsigned integer |
| `ReadUInt64(address)` | `ulong?` | Read 8-byte unsigned integer |
| `ReadBool(address)` | `bool?` | Read boolean (1 byte) |
| `ReadPointer(address)` | `long?` | Read 8-byte pointer |

### Strings

| Method | Return Type | Description |
|--------|-------------|-------------|
| `ReadString(address, maxLength=256)` | `string?` | Read null-terminated ASCII string |
| `ReadUnicodeString(address, maxLength=256)` | `string?` | Read null-terminated UTF-16 string |

```csharp
// Read various types
int? health = ReadInt32(healthAddress);
float? posX = ReadFloat(playerBase + 0x10);
string? name = ReadString(nameAddress);

// Null-safety with nullable types
if (health.HasValue) {
    Print($"Health: {health.Value}");
}
```

---

## Memory Write Operations

### Basic Types

| Method | Return Type | Description |
|--------|-------------|-------------|
| `WriteByte(address, value)` | `bool` | Write 1 byte |
| `WriteBytes(address, data)` | `bool` | Write byte array |
| `WriteInt32(address, value)` | `bool` | Write 4-byte signed integer |
| `WriteInt64(address, value)` | `bool` | Write 8-byte signed integer |
| `WriteFloat(address, value)` | `bool` | Write 4-byte float |
| `WriteDouble(address, value)` | `bool` | Write 8-byte double |
| `WriteBool(address, value)` | `bool` | Write boolean (1 byte) |

### Strings

| Method | Return Type | Description |
|--------|-------------|-------------|
| `WriteString(address, value, nullTerminate=true)` | `bool` | Write ASCII string |
| `WriteUnicodeString(address, value, nullTerminate=true)` | `bool` | Write UTF-16 string |

```csharp
// Set health to 9999
WriteInt32(healthAddress, 9999);

// Set position
WriteFloat(posX, 100.0f);
WriteFloat(posY, 50.0f);

// Write player name
WriteUnicodeString(nameAddress, "Hacker");
```

---

## Extended Memory Types

### Arrays

| Method | Return Type | Description |
|--------|-------------|-------------|
| `ReadInt32Array(address, count)` | `int[]?` | Read array of int32 |
| `ReadInt64Array(address, count)` | `long[]?` | Read array of int64 |
| `ReadFloatArray(address, count)` | `float[]?` | Read array of floats |
| `ReadDoubleArray(address, count)` | `double[]?` | Read array of doubles |
| `WriteInt32Array(address, values)` | `bool` | Write array of int32 |
| `WriteFloatArray(address, values)` | `bool` | Write array of floats |

### Vectors

| Method | Return Type | Description |
|--------|-------------|-------------|
| `ReadVector2(address)` | `(float X, float Y)?` | Read 2D vector |
| `ReadVector3(address)` | `(float X, float Y, float Z)?` | Read 3D vector |
| `ReadVector4(address)` | `(float X, float Y, float Z, float W)?` | Read 4D vector/quaternion |
| `WriteVector3(address, x, y, z)` | `bool` | Write 3D vector |

```csharp
// Read player position as vector
var pos = ReadVector3(playerBase + 0x100);
if (pos.HasValue) {
    Print($"Position: ({pos.Value.X}, {pos.Value.Y}, {pos.Value.Z})");
}

// Teleport player
WriteVector3(playerBase + 0x100, 1000.0f, 500.0f, 0.0f);

// Read inventory array
int[]? inventory = ReadInt32Array(inventoryBase, 10);
```

---

## AOB/Pattern Scanning

Scan process memory for byte patterns with wildcard support.

| Method | Return Type | Description |
|--------|-------------|-------------|
| `AOBScan(pattern, startAddr=0, endAddr=0)` | `List<long>` | Find all matches |
| `AOBScanFirst(pattern, startAddr=0, endAddr=0)` | `long?` | Find first match |
| `AOBScanModule(pattern, moduleName)` | `List<long>` | Scan specific module |

### Pattern Format

- Hex bytes separated by spaces: `"48 8B 05 10 20 30 40"`
- Wildcards: `??`, `?`, `*`, `**`
- Example: `"48 8B ?? ?? ?? ?? ?? 48 89"`

```csharp
// Find health decrease instruction
var pattern = "29 87 ?? ?? 00 00 8B 87";  // sub [rdi+????], eax
var results = AOBScan(pattern);
Print($"Found {results.Count} matches");

// Get first result
long? addr = AOBScanFirst("48 8B 05 ?? ?? ?? ?? 48 85");
if (addr.HasValue) {
    Print($"Found at: 0x{addr.Value:X}");
}

// Scan only game.exe module
var matches = AOBScanModule("F3 0F 10 ?? ?? 00 00 00", "game.exe");
```

---

## Patch Helpers

Write common instruction patterns for code modification.

| Method | Return Type | Description |
|--------|-------------|-------------|
| `Nop(address, count, saveName=null)` | `byte[]?` | Write NOP instructions (0x90) |
| `WriteJmp(from, to, saveName=null)` | `byte[]?` | Write 5-byte JMP (E9) |
| `WriteCall(from, to, saveName=null)` | `byte[]?` | Write 5-byte CALL (E8) |
| `WriteRet(address)` | `bool` | Write RET instruction (C3) |
| `CreateCodeCave(hookAddr, caveCode, hookSize=5)` | `long?` | Create code cave with hook |
| `Assemble(instruction)` | `byte[]?` | Assemble simple instruction |

### Assembler Support

The `Assemble()` method supports basic x64 instructions:
- `NOP`, `RET`, `INT3`
- `XOR reg, reg` (zero register)
- `MOV reg, imm32`

```csharp
// NOP out damage calculation (6 bytes)
Nop(damageAddr, 6);

// Hook function to custom code
WriteJmp(originalFunc, myCodeCave);

// Make function return immediately
WriteRet(annoyingFunc);

// Assemble instructions
byte[]? nop = Assemble("NOP");
byte[]? zero = Assemble("XOR EAX, EAX");
byte[]? loadVal = Assemble("MOV EAX, 0x100");
```

---

## Save/Restore System

Persist original bytes for clean enable/disable toggling in scripts.

| Method | Return Type | Description |
|--------|-------------|-------------|
| `SaveOriginalBytes(name, address, count)` | `bool` | Save bytes with a name |
| `RestoreOriginalBytes(name)` | `bool` | Restore saved bytes |
| `GetSavedBytes(name)` | `byte[]?` | Get saved bytes without restoring |
| `RestoreAllBytes()` | `void` | Restore all saved bytes |

### Usage with Patch Helpers

The `Nop`, `WriteJmp`, and `WriteCall` methods accept an optional `saveName` parameter:

```csharp
#if ENABLED
// These automatically save original bytes
Nop(0x12345678, 6, "damage_patch");
WriteJmp(hookAddr, caveAddr, "damage_hook");
#endif

#if DISABLED
// Restore all patches
RestoreAllBytes();
#endif
```

### Manual Save/Restore

```csharp
#if ENABLED
// Save before modifying
SaveOriginalBytes("my_patch", 0x12345678, 10);
WriteBytes(0x12345678, new byte[] { 0x90, 0x90, 0x90, 0x90, 0x90 });
#endif

#if DISABLED
RestoreOriginalBytes("my_patch");
#endif
```

---

## Memory Allocation

Allocate memory in the target process for code caves and data.

| Method | Return Type | Description |
|--------|-------------|-------------|
| `AllocateMemory(size, executable=false)` | `long?` | Allocate memory |
| `Alloc(size)` | `long?` | Allocate executable memory (shorthand) |
| `FreeMemory(address)` | `bool` | Free allocated memory |
| `FreeAllMemory()` | `void` | Free all allocated memory |
| `SetMemoryProtection(address, size, executable)` | `bool` | Change memory protection |

```csharp
// Allocate code cave
long? cave = Alloc(256);
if (cave.HasValue) {
    // Write custom code
    WriteBytes(cave.Value, new byte[] {
        0x48, 0x89, 0xC1,  // mov rcx, rax
        0xC3               // ret
    });

    // Hook original function
    WriteJmp(originalFunc, cave.Value);
}

// Cleanup
FreeAllMemory();
```

---

## Pointer Operations

Follow pointer chains and resolve complex addresses.

| Method | Return Type | Description |
|--------|-------------|-------------|
| `ReadPointer(address)` | `long?` | Read 8-byte pointer |
| `ReadPointerChain(baseAddr, offsets...)` | `long?` | Follow pointer chain |1

```csharp
// Follow pointer chain: [[base+0x10]+0x20]+0x30
long? finalAddr = ReadPointerChain(baseAddress, 0x10, 0x20, 0x30);
if (finalAddr.HasValue) {
    int? value = ReadInt32(finalAddr.Value);
}

// Manual pointer following
long? ptr1 = ReadPointer(baseAddress);
long? ptr2 = ReadPointer(ptr1.Value + 0x10);
long? result = ReadPointer(ptr2.Value + 0x20);
```

---

## Module Helpers

Work with loaded modules (DLLs, EXEs).

| Method | Return Type | Description |
|--------|-------------|-------------|
| `GetModuleBase(moduleName)` | `long?` | Get module base address |
| `ResolveAddress(addressString)` | `long?` | Parse module+offset format |
| `GetRegions()` | `List<MemoryRegionInfo>` | Get all memory regions |

### Address Formats

`ResolveAddress` supports:
- Module+offset: `"game.exe+0x123456"`, `"UnityPlayer.dll+1A2B3C"`
- Plain hex: `"0x7FF612340000"`, `"7FF612340000"`

```csharp
// Get module base
long? gameBase = GetModuleBase("game.exe");
long? unityBase = GetModuleBase("UnityPlayer.dll");

// Resolve address from string
long? addr = ResolveAddress("game.exe+0x123456");

// Enumerate memory regions
foreach (var region in GetRegions()) {
    Print($"0x{region.BaseAddress:X} - {region.RegionSize} bytes");
}
```

---

## Freeze API

Add frozen values to the address list that continuously write a value.

| Method | Return Type | Description |
|--------|-------------|-------------|
| `FreezeInt32(address, value, description="")` | `bool` | Freeze int32 value |
| `FreezeInt64(address, value, description="")` | `bool` | Freeze int64 value |
| `FreezeFloat(address, value, description="")` | `bool` | Freeze float value |
| `FreezeDouble(address, value, description="")` | `bool` | Freeze double value |
| `FreezeByte(address, value, description="")` | `bool` | Freeze byte value |
| `Unfreeze(address)` | `bool` | Remove freeze from address |
| `UnfreezeAll()` | `void` | Remove all script freezes |

```csharp
#if ENABLED
// Freeze health at 9999
FreezeInt32(healthAddress, 9999, "Infinite Health");

// Freeze ammo
FreezeInt32(ammoAddress, 999, "Infinite Ammo");
#endif

#if DISABLED
// Clean up
UnfreezeAll();
#endif
```

---

## Breakpoint API

Set hardware breakpoints for debugging and value interception.

### Initialization

| Method | Return Type | Description |
|--------|-------------|-------------|
| `InitializeBreakpoints()` | `bool` | Initialize debugger |
| `SetDebuggerMode(mode)` | `void` | Set debugger mode |
| `CleanupBreakpoints()` | `void` | Cleanup debugger |

### Debugger Modes

- `"debug"` - Windows Debug API (default)
- `"veh"` - VEH injection (for anti-debug games)
- `"kernel"` - Kernel driver (dbk64.sys)

### Setting Breakpoints

| Method | Return Type | Description |
|--------|-------------|-------------|
| `SetBreakpoint(address, type)` | `bool` | Set breakpoint (max 4) |
| `SetBreakpoint(address, callback, type)` | `bool` | Set breakpoint with callback |
| `RemoveBreakpoint(address)` | `bool` | Remove breakpoint |

### Breakpoint Types

| Type | Description |
|------|-------------|
| `BreakpointType.Execute` | Break when instruction executes |
| `BreakpointType.Write` | Break when memory is written |
| `BreakpointType.ReadWrite` | Break on read or write |

### Register Access

| Method | Return Type | Description |
|--------|-------------|-------------|
| `WaitForBreakpoint(timeoutMs=5000)` | `RegisterState?` | Wait for breakpoint hit |
| `GetLastRegisters()` | `RegisterState?` | Get last captured registers |

### RegisterState Properties

General-purpose: `RAX`, `RBX`, `RCX`, `RDX`, `RSI`, `RDI`, `RBP`, `RSP`, `R8`-`R15`, `RIP`, `RFLAGS`

XMM methods:
- `GetXMMFloat(index, floatIndex=0)` - Get float from XMM register
- `SetXMMFloat(index, value, floatIndex=0)` - Set float in XMM register
- `GetXMMDouble(index, doubleIndex=0)` - Get double from XMM register
- `SetXMMDouble(index, value, doubleIndex=0)` - Set double in XMM register

```csharp
#if ENABLED
// Callback method - runs on every breakpoint hit
int OnDamage(RegisterState regs)
{
    float damage = regs.GetXMMFloat(0);
    regs.SetXMMFloat(0, 0.0f);  // No damage
    return 0;  // Continue execution
}

InitializeBreakpoints();
SetBreakpoint(damageCalcAddr, OnDamage, BreakpointType.Execute);
#endif

#if DISABLED
CleanupBreakpoints();
#endif
```

---

## Managed Runtime Support

Introspect Mono and .NET applications (Unity games, .NET apps).

### Connection

| Method | Return Type | Description |
|--------|-------------|-------------|
| `DetectRuntime()` | `string` | Detect runtime type |
| `ConnectToRuntime()` | `Task<bool>` | Connect to runtime |
| `DisconnectRuntime()` | `void` | Disconnect |

### Runtime Types

- `"Mono"` - Unity (Mono runtime)
- `"DotNetCore"` - .NET 5/6/7/8
- `"DotNetFramework"` - .NET Framework 4.x
- `"None"` - No managed runtime

### Type Enumeration

| Method | Return Type | Description |
|--------|-------------|-------------|
| `GetDomains()` | `Task<List<ManagedDomainInfo>>` | Get app domains |
| `GetAssemblies(domainHandle=0)` | `Task<List<ManagedAssemblyInfo>>` | Get assemblies |
| `GetTypes(imageHandle)` | `Task<List<ManagedTypeInfo>>` | Get types in assembly |
| `FindType(fullName)` | `Task<ManagedTypeInfo?>` | Find type by name |
| `GetTypeDetails(typeHandle)` | `Task<ManagedTypeInfo?>` | Get type fields/methods |

### Instance Operations

| Method | Return Type | Description |
|--------|-------------|-------------|
| `GetStaticFieldAddress(typeHandle, fieldHandle)` | `Task<long>` | Get static field address |
| `FindInstances(typeHandle, maxResults=100)` | `Task<List<long>>` | Find heap instances (.NET only) |

```csharp
// Detect and connect
string runtime = DetectRuntime();
Print($"Runtime: {runtime}");

if (await ConnectToRuntime()) {
    // Find player type
    var playerType = await FindType("PlayerController");
    if (playerType != null) {
        Print($"Found: {playerType.FullName}");

        // Get details
        var details = await GetTypeDetails(playerType.TypeHandle);
        foreach (var field in details.Fields) {
            Print($"  {field.TypeName} {field.Name} @ +0x{field.Offset:X}");
        }
    }

    // Find all instances (.NET only)
    var instances = await FindInstances(playerType.TypeHandle);
    Print($"Found {instances.Count} player instances");
}
```

---

## IL2CPP Support

Work with Unity IL2CPP games - parse metadata and resolve runtime addresses.

### Metadata Loading

| Method | Return Type | Description |
|--------|-------------|-------------|
| `LoadIL2CppMetadata()` | `bool` | Auto-detect and load global-metadata.dat |
| `LoadIL2CppMetadata(path)` | `bool` | Load from specific path |
| `FindIL2CppType(fullName)` | `IL2CppTypeInfo?` | Find type by name |
| `SearchIL2CppTypes(pattern)` | `List<IL2CppTypeInfo>` | Search types by pattern |
| `GetIL2CppImages()` | `List<IL2CppImageInfo>` | Get all assemblies |

### Runtime Address Resolution

After loading metadata, connect to the runtime to get actual addresses:

| Method | Return Type | Description |
|--------|-------------|-------------|
| `ConnectIL2CppRuntime()` | `bool` | Connect to GameAssembly.dll |
| `GetIL2CppMethodAddress(index)` | `long` | Get method address by index |
| `GetIL2CppMethodAddress(type, method)` | `long` | Get method address by name |
| `GetGameAssemblyBase()` | `long` | Get GameAssembly.dll base |
| `GetIL2CppExport(name)` | `long` | Get IL2CPP API export |
| `GetIL2CppExports()` | `Dictionary<string, long>` | Get all IL2CPP exports |
| `FindIL2CppStaticField(type, field)` | `long` | Find static field address |
| `ResolveIL2CppMethods(typeName)` | `void` | Resolve all methods for type |

### Complete IL2CPP Workflow

```csharp
// Step 1: Load metadata (auto-detects path)
if (!LoadIL2CppMetadata()) {
    Print("Failed to load metadata");
    return;
}
// Or specify path manually:
// LoadIL2CppMetadata(@"C:\Game\Data\il2cpp_data\Metadata\global-metadata.dat");

// Step 2: Connect to runtime (gets actual addresses from GameAssembly.dll)
if (!ConnectIL2CppRuntime()) {
    Print("Failed to connect to runtime");
    return;
}

// Step 3: Find and use methods
var playerType = FindIL2CppType("PlayerController");
if (playerType != null) {
    // Resolve all method addresses for this type
    ResolveIL2CppMethods("PlayerController");

    // Get specific method address
    long takeDamageAddr = GetIL2CppMethodAddress("PlayerController", "TakeDamage");
    if (takeDamageAddr != 0) {
        Print($"TakeDamage at: 0x{takeDamageAddr:X}");

        // Hook it!
        Nop(takeDamageAddr, 6, "damage_patch");
    }
}

// Step 4: Use IL2CPP API exports
long domainGet = GetIL2CppExport("il2cpp_domain_get");
Print($"il2cpp_domain_get at: 0x{domainGet:X}");

// List all exports
foreach (var export in GetIL2CppExports()) {
    Print($"{export.Key} -> 0x{export.Value:X}");
}
```

### Finding Static Fields (Recommended Approach)

The recommended way to access IL2CPP static fields is via `ConnectToRuntime()`. This approach works for both Mono and IL2CPP games and provides accurate field offsets.

```csharp
// Step 1: Connect to runtime (works for both Mono and IL2CPP)
if (!await ConnectToRuntime()) {
    Print("Failed to connect to runtime");
    return;
}

// Step 2: Find the type
var type = await FindType("Assets.Scripts.Simulation.TimeManager");
if (type == null) {
    Print("Type not found");
    return;
}
Print($"Found type: {type.FullName} (handle: 0x{type.TypeHandle:X})");

// Step 3: Get type details with fields
var details = await GetTypeDetails(type.TypeHandle);
Print($"Found {details.Fields.Count} fields:");
foreach (var field in details.Fields) {
    string staticTag = field.IsStatic ? " [STATIC]" : "";
    Print($"  {field.TypeName} {field.Name} @ +0x{field.Offset:X}{staticTag}");
}

// Step 4: Get static field address
var targetField = details.Fields.FirstOrDefault(f => f.Name == "replayTimeScaleMultiplier");
if (targetField != null && targetField.IsStatic) {
    long addr = await GetStaticFieldAddress(type.TypeHandle, targetField.FieldHandle);
    if (addr != 0) {
        Print($"Static field address: 0x{addr:X}");

        // IMPORTANT: Use the correct Read/Write method based on field type!
        // For System.Single (float) use ReadFloat/WriteFloat
        // For System.Double use ReadDouble/WriteDouble
        // For System.Int32 use ReadInt32/WriteInt32

        if (targetField.TypeName == "System.Double" || targetField.TypeName == "double") {
            double value = ReadDouble(addr) ?? 0;
            Print($"Current value: {value}");
            WriteDouble(addr, 2.0);  // Set to 2x speed
        } else {
            float value = ReadFloat(addr) ?? 0;
            Print($"Current value: {value}");
            WriteFloat(addr, 2.0f);
        }
    }
}
```

### Alternative: FindIL2CppStaticField (Metadata-only)

For cases where you only need metadata parsing without runtime connection:

```csharp
// Find static field using metadata parser (less reliable)
long instanceAddr = FindIL2CppStaticField("GameManager", "Instance");
if (instanceAddr != 0) {
    long instancePtr = ReadInt64(instanceAddr) ?? 0;
    Print($"GameManager.Instance at: 0x{instancePtr:X}");

    // Read fields from the instance
    var gameManagerType = FindIL2CppType("GameManager");
    foreach (var field in gameManagerType.Fields) {
        Print($"  {field.Name} @ +0x{field.Offset:X}");
    }
}
```

### Hooking IL2CPP Methods with Breakpoints

Use breakpoint callbacks to intercept and modify method behavior. Define callback methods with signature `int MethodName(RegisterState regs)`.

```csharp
#if ENABLED
int NullifyDamage(RegisterState regs)
{
    // XMM1 often contains float parameters
    regs.SetXMMFloat(1, 0.0f);
    return 0;
}

int DamageMultiplier(RegisterState regs)
{
    float damage = regs.GetXMMFloat(1);
    regs.SetXMMFloat(1, damage * 10.0f);
    return 0;
}

LoadIL2CppMetadata();  // Auto-detects path
ConnectIL2CppRuntime();
InitializeBreakpoints();

long playerDamage = GetIL2CppMethodAddress("Player", "TakeDamage");
SetBreakpoint(playerDamage, NullifyDamage, BreakpointType.Execute);

long enemyDamage = GetIL2CppMethodAddress("Enemy", "TakeDamage");
SetBreakpoint(enemyDamage, DamageMultiplier, BreakpointType.Execute);
#endif

#if DISABLED
CleanupBreakpoints();
#endif
```

### Reading Object Fields in Breakpoint

```csharp
#if ENABLED
LoadIL2CppMetadata();  // Auto-detects path
ConnectIL2CppRuntime();

var playerType = FindIL2CppType("Player");
var healthField = playerType?.Fields.FirstOrDefault(f => f.Name == "health");
int healthOffset = healthField?.Offset ?? 0x20;

int InfiniteHealth(RegisterState regs)
{
    // RCX = 'this' pointer in x64 calling convention
    long playerPtr = regs.RCX;
    float health = ReadFloat(playerPtr + healthOffset) ?? 0;

    if (health < 100) {
        WriteFloat(playerPtr + healthOffset, 100.0f);
    }
    return 0;
}

long updateAddr = GetIL2CppMethodAddress("Player", "Update");
if (updateAddr != 0) {
    InitializeBreakpoints();
    SetBreakpoint(updateAddr, InfiniteHealth, BreakpointType.Execute);
}
#endif

#if DISABLED
CleanupBreakpoints();
#endif
```

### IL2CPP Data Types

**IL2CppTypeInfo:**
- `Index` - Type index in metadata
- `Name` / `FullName` - Type name
- `Namespace` - Type namespace
- `Fields` - List of IL2CppFieldInfo
- `Methods` - List of IL2CppMethodInfo

**IL2CppMethodInfo:**
- `Index` - Method index (use with GetIL2CppMethodAddress)
- `Name` - Method name
- `NativeAddress` - Resolved native address (after ConnectIL2CppRuntime)
- `ParameterCount` - Number of parameters

**IL2CppFieldInfo:**
- `Index` - Field index
- `Name` - Field name
- `TypeIndex` - Field type index
- `Offset` - Field offset (resolved via runtime)
- `IsOffsetResolved` - Whether offset has been resolved

---

## Script Sections

Scripts support enable/disable sections for trainer-style toggles.

```csharp
#if ENABLED
// Code runs when script is enabled
SaveOriginalBytes("damage", damageAddr, 6);
Nop(damageAddr, 6);
Print("Damage nullifier active!");
#endif

#if DISABLED
// Code runs when script is disabled
RestoreOriginalBytes("damage");
Print("Damage nullifier disabled!");
#endif
```

---

## Complete Examples

### Example 1: No Damage Script

```csharp
#if ENABLED
// Find damage function
var pattern = "29 87 ?? ?? 00 00";  // sub [rdi+????], eax
long? damageAddr = AOBScanFirst(pattern);

if (damageAddr.HasValue) {
    Nop(damageAddr.Value, 6, "damage_patch");
    Print($"Damage disabled at 0x{damageAddr.Value:X}");
} else {
    Print("ERROR: Pattern not found");
}
#endif

#if DISABLED
RestoreAllBytes();
Print("Damage restored");
#endif
```

### Example 2: Infinite Ammo with Freeze

```csharp
#if ENABLED
// Resolve ammo address
long? ammoAddr = ResolveAddress("game.exe+0x123456");
if (ammoAddr.HasValue) {
    // Follow pointer chain
    long? finalAddr = ReadPointerChain(ammoAddr.Value, 0x10, 0x20, 0x8);
    if (finalAddr.HasValue) {
        FreezeInt32(finalAddr.Value, 999, "Infinite Ammo");
    }
}
#endif

#if DISABLED
UnfreezeAll();
#endif
```

### Example 3: Damage Multiplier with Breakpoint

```csharp
#if ENABLED
int MultiplyDamage(RegisterState regs)
{
    float damage = regs.GetXMMFloat(0);
    regs.SetXMMFloat(0, damage * 10.0f);
    return 0;
}

long? damageCalc = AOBScanFirst("F3 0F 59 ?? ?? ?? 00 00");
if (damageCalc.HasValue) {
    InitializeBreakpoints();
    SetBreakpoint(damageCalc.Value, MultiplyDamage, BreakpointType.Execute);
    Print("Damage multiplier active!");
}
#endif

#if DISABLED
CleanupBreakpoints();
#endif
```

### Example 4: Unity/Mono Type Discovery

```csharp
// Detect and connect to runtime
string runtime = DetectRuntime();
Print($"Detected: {runtime}");

if (runtime == "Mono" || runtime == "DotNetCore") {
    if (await ConnectToRuntime()) {
        // List all assemblies
        var assemblies = await GetAssemblies();
        foreach (var asm in assemblies) {
            Print($"Assembly: {asm.Name}");
        }

        // Find specific type
        var playerType = await FindType("Player");
        if (playerType != null) {
            var details = await GetTypeDetails(playerType.TypeHandle);
            Print($"\nFields in {details.FullName}:");
            foreach (var f in details.Fields) {
                Print($"  +0x{f.Offset:X2} {f.TypeName} {f.Name}");
            }
        }
    }
}
```

### Example 5: Code Cave for Custom Logic

```csharp
#if ENABLED
// Create code cave for health check
byte[] caveCode = new byte[] {
    0x83, 0xF8, 0x00,       // cmp eax, 0
    0x7F, 0x05,             // jg skip
    0xB8, 0x01, 0x00, 0x00, 0x00,  // mov eax, 1 (minimum health)
};

long? hookAddr = AOBScanFirst("89 87 ?? ?? 00 00");  // mov [rdi+????], eax
if (hookAddr.HasValue) {
    SaveOriginalBytes("health_cave", hookAddr.Value, 6);
    long? cave = CreateCodeCave(hookAddr.Value, caveCode, 6);
    if (cave.HasValue) {
        Print($"Code cave at 0x{cave.Value:X}");
    }
}
#endif

#if DISABLED
RestoreAllBytes();
FreeAllMemory();
#endif
```

---

## Output and Debugging

| Method | Description |
|--------|-------------|
| `Print(message)` | Output to script console |
| `GotoAddress(address)` | Navigate Memory Browser to address |
| `ShowBreakpointLog()` | Display debugger log |
| `DumpClassStructure(typeHandle)` | Dump IL2CPP class structure for debugging |
| `DumpStaticFields(baseAddr, size=64)` | Dump raw bytes from static fields area |

```csharp
Print("Script started!");
Print($"Base address: 0x{GetModuleBase("game.exe"):X}");

// Navigate to address in Memory Browser
GotoAddress(0x7FF612340000);
```

### IL2CPP Debugging

When troubleshooting IL2CPP static field access:

```csharp
// Dump class structure to understand offsets
var type = await FindType("GameManager");
if (type != null) {
    DumpClassStructure(type.TypeHandle);
}

// Get static field base and dump raw bytes
long staticBase = await GetStaticFieldAddress(type.TypeHandle, 0);
if (staticBase != 0) {
    DumpStaticFields(staticBase, 128);  // Dump 128 bytes
}
```

---

## Error Handling

All read operations return nullable types. Check for `null` before using:

```csharp
int? value = ReadInt32(address);
if (value == null) {
    Print("Failed to read memory!");
    return;
}
Print($"Value: {value.Value}");

// Or use null-coalescing
int health = ReadInt32(address) ?? 0;
```

Write operations return `bool`:

```csharp
if (!WriteInt32(address, 9999)) {
    Print("Failed to write memory!");
}
```

---

## Notes

- All addresses are `long` (64-bit)
- Memory operations require the process to be attached
- Hardware breakpoints are limited to 4 (CPU DR0-DR3 registers)
- Use `ConnectToRuntime()` for static field access - works for both Mono and IL2CPP games
- **Important**: Match Read/Write methods to actual field types:
  - `System.Single` / `float` → `ReadFloat()` / `WriteFloat()`
  - `System.Double` / `double` → `ReadDouble()` / `WriteDouble()`
  - `System.Int32` / `int` → `ReadInt32()` / `WriteInt32()`
- Breakpoint callbacks use signature `int MethodName(RegisterState regs)` - return 0 to continue
- Scripts persist between toggles, so save/restore system maintains state
