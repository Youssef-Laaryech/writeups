# ilspycmd

## What it is

`ilspycmd` is the command-line version of **ILSpy**, an open-source .NET decompiler. It takes a `.NET` executable or DLL and reconstructs the original C# source code from the compiled IL (Intermediate Language) bytecode.

It runs cross-platform (Linux, macOS, Windows) and is the go-to tool for .NET reverse engineering on a Linux pentest machine.

## Installation

Requires the .NET SDK (version ≥ 6.0):

```bash
# Install .NET SDK first if not present
sudo apt install dotnet-sdk-8.0

# Install ilspycmd as a global tool
dotnet tool install -g ilspycmd

# Add to PATH
echo 'export PATH="$PATH:$HOME/.dotnet/tools"' >> ~/.bashrc
source ~/.bashrc

# Pin to a specific version (e.g. to match .NET 6.0 SDK)
dotnet tool install -g ilspycmd --version 7.2.1.6856
```

## Basic Usage

```bash
# Decompile a single binary to a directory of .cs files
ilspycmd UserInfo.exe -o decompiled/

# Decompile and print to stdout
ilspycmd UserInfo.exe

# List all types/classes in the binary
ilspycmd UserInfo.exe --list-all

# Decompile a specific type
ilspycmd UserInfo.exe -t Protected
```

## What the Output Looks Like

```
decompiled/
├── UserInfo/
│   ├── Program.cs
│   ├── Commands/
│   │   ├── FindCommand.cs
│   │   └── UserCommand.cs
│   └── Protected.cs    ← where the obfuscated password is
```

## What to Look For After Decompiling

```bash
# Hardcoded credentials
grep -ri "password\|secret\|key\|token" decompiled/

# LDAP / AD connection strings
grep -ri "LDAP://\|DirectoryEntry\|DirectorySearcher" decompiled/

# Obfuscation / encryption routines
grep -ri "Convert.FromBase64\|XOR\|0xDF\|armando" decompiled/

# Network calls
grep -ri "HttpClient\|WebClient\|WebRequest" decompiled/
```

## Alternative: dnSpy (Windows only)

If you have access to a Windows VM, **dnSpy** provides a GUI with full debugging support — you can set breakpoints, inspect runtime values, and even edit + recompile the binary live. Useful when static analysis alone isn't enough.

## Related

- [[dotNET Binary Reverse Engineering]]
- [[Credential Reuse]]
