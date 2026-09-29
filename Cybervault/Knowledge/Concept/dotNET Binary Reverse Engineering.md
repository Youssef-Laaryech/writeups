# .NET Binary Reverse Engineering

## Why .NET Binaries Are Reversible

.NET executables (`.exe`, `.dll`) compile to **Intermediate Language (IL / CIL)** bytecode rather than native machine code. IL retains:

- Class and method names
- Variable types and names
- Full method signatures
- String literals

This means a decompiler can reconstruct **near-perfect, readable C# source code** from the binary — unlike compiled C/C++ where decompilation yields hard-to-read assembly and pseudo-C.

## Identifying a .NET Binary

Signs a binary is .NET:

- Presence of `.dll` files alongside the `.exe` with names like:
  - `System.Runtime.dll`
  - `Microsoft.Extensions.*.dll`
  - `CommandLineParser.dll`
  - `Newtonsoft.Json.dll`
- Running `file UserInfo.exe` on Linux → `PE32 executable (console) Intel 80386 Mono/.Net assembly`
- On Windows: right-click properties → under "Details", see .NET Framework version

## Tools

| Tool | Platform | Notes |
|------|----------|-------|
| `ilspycmd` | Linux / macOS CLI | Command-line ILSpy — outputs decompiled C# to files |
| ILSpy | Windows GUI | Full GUI decompiler, browse class tree |
| dnSpy | Windows GUI | GUI + debugger — can set breakpoints, modify and recompile |
| dotPeek | Windows GUI | JetBrains free decompiler |

## ilspycmd (Linux)

Install:

```bash
dotnet tool install -g ilspycmd --version 7.2.1.6856
export PATH="$PATH:$HOME/.dotnet/tools"
```

Decompile to directory:

```bash
mkdir decompiled
ilspycmd UserInfo.exe -o decompiled/
```

Browse the output directory — each namespace becomes a folder, each class a `.cs` file.

## What to Look For

After decompiling, search for:

```bash
# Hardcoded credentials
grep -ri "password\|passwd\|secret\|token\|key\|ldap" decompiled/

# Connection strings / URLs
grep -ri "http\|ldap\|ftp\|smb\|\\\\\\\\192\|@domain" decompiled/

# Encryption / obfuscation routines
grep -ri "XOR\|base64\|Convert.From\|Encoding\." decompiled/
```

## Common Obfuscation Patterns

### XOR with a hardcoded key

```csharp
byte[] key = Encoding.ASCII.GetBytes("armando");
for (int i = 0; i < array.Length; i++)
    array[i] = (byte)((array[i] ^ key[i % key.Length]) ^ 0xDF);
```

**Reverse in Python:**

```python
import base64

enc = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E"
key = b"armando"
data = base64.b64decode(enc)
print(bytes((b ^ key[i % len(key)]) ^ 0xDF for i, b in enumerate(data)).decode("latin-1"))
```

### Base64 encoded string

```python
import base64
print(base64.b64decode("aGVsbG8=").decode())
```

### ROT13

```python
import codecs
print(codecs.decode("uryyb", "rot_13"))
```

## Related

- [[Credential Reuse]]
- [[smb,smbmap, smbcclient]]
