# Large Address Aware (aka 4GB patch) autopatcher

This script automatically finds your game libraries (Steam, GOG, Epic) and patches all 32‑bit `.exe` files to be **Large Address Aware** (LAA). This allows 32‑bit games to use more than 2 GB of RAM, improving stability and performance.

## Patching tool support

- **[patchlaa](https://docs.rs/crate/patchlaa/latest)** (recommended) - modern, fast, written in Rust (≈98% success rate).  
- **[4GB Patch](https://ntcore.com/4gb-patch/)** - old GUI/CLI tool that doesn't really work well (included for legacy reasons).  
- **[editbin](https://visualstudio.microsoft.com/downloads/)** - Microsoft tool from Visual Studio, very slow but has a perfect success rate.

## Features

- Auto‑detects Steam, GOG and Epic libraries.  
- Support for custom libraries (e.g., `D:\Games`, `E:\SteamLibrary`).  
- Ignores specific folders or file names to avoid patching non‑game files (e.g., `*setup.exe`, `DirectX`, `Tools`).  
- Supports multiple patchers (`patchlaa`, `4GB Patch`, `editbin`).  
- Option to create backups of the eligible `.exe` files before patching.
- Verbose and extensive logging of everything that happens.

## Dependencies

- All patchers require **Windows 7+** with elevated (admin) permissions.  
- **PowerShell 5.1+** (obviously).

### For `patchlaa`
- **Rust** and **Cargo** are automatically installed by the script if missing (via `winget`).

### For `editbin`
- **[Visual Studio](https://visualstudio.microsoft.com/downloads/)** (any edition) with the C++ workload.

## Installation & Usage

1. Download the script and run it **as Administrator** (the script will self‑elevate if possible).  

   Or simply run the following command in powershell:
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass; irm "https://raw.githubusercontent.com/idontknowshark/4GB-AutoPatcher/refs/heads/main/LAA-AutoPatcher.ps1" | iex
```
2. Choose a patching tool.  
3. The script auto‑detects your game libraries. You can choose to include or skip each one.  
4. Optionally add extra folders (e.g., `D:\GOG Games`, `C:\MyGames`).  
5. Optionally specify patterns to skip (e.g., `Mods`, `_CommonRedist`, `*launcher.exe`).  
6. The script scans all `.exe` files, filters by architecture (only 32‑bit are patched), and runs the chosen patcher on each.  
7. A summary shows how many files were patched, skipped, or failed.

## Patcher comparison

| Patcher       | Success rate (tested) | Speed                   | Dependencies                 | Notes                                                       |
|---------------|-----------------------|-------------------------|------------------------------|-------------------------------------------------------------|
| `patchlaa`    | ≈98%                  | <0.1 seconds per file   | Rust                         | **Recommended** - modern, gives clear output.               |
| `4gb_patch`   | None                  | <0.1 seconds per file   | None                         | GUI tool; CLI mode often fails.                             |
| `editbin`     | 100%                  | ~1-2 seconds per file   | Visual Studio                | Official Microsoft tool; requires large download.           |

### Why not always use `editbin`?
Because it requires the user to install Visual Studio Build Tools, which is a huge download and overkill for most people. 
It is also a lot slower than `patchlaa`, which gives nearly the same success rate with a tiny footprint.

## How it works

- The script reads the PE header of every `.exe` it finds.  
- If the machine type is `0x14C` (32‑bit), it applies the Large Address Aware flag and runs the chosen patcher on the `.exe`.  
- 64‑bit executables (`0x8664`) are skipped because they already support more than 2 GB.  
- It respects your ignore patterns - so it never patches installers, redistributables, anti‑cheat modules, etc.

## Testing results

The script was tested on a real system with **1192 total `.exe` files** across three game folders (Steam default, a secondary Steam library on `D:\`, and a custom `D:\Games` folder). Not tested with GOG and Epic libraries because I do not have them.

- **32‑bit executables found:** 512  
- **64‑bit executables:** 680 (correctly skipped)

### With `patchlaa` 
- **Successfully patched:** 503 out of 512 (≈98%)  
- **Failed:** 9 files (all due to malformed PE headers - the patcher crashed with a `Malformed("ResourceString value_len...")` error).  

### With `editbin` (Microsoft tool)
- **Successfully patched:** 512 out of 512 (100%)  
- **No failures** - every 32‑bit executable was patched, including the 9 that crashed `patchlaa`.  
- **Drawback:** Patching was noticeably slower (~1 second per file vs. <0.1 s with `patchlaa`)

### With `4GB Patch` (CLI mode)
- **Initial runs:** All patches failed silently with exit code 1 and no output.  
- **After adding elevation and using `Start-Process`:** Some files were patched successfully, but the tool often spawned an interactive file‑picker dialog, making automation impossible.  
- **Conclusion:** The 4GB Patch’s command‑line mode is fundamentally broken and **not suitable for scripting**.

### Bottom line
- For most users, `patchlaa` is the best choice - it’s fast, lightweight, and succeeds on all but a few corrupted executables.  
- For the few problematic files (or if you need 100% success), `editbin` is a reliable fallback, at the cost of a large dependency.  
- The 4GB Patch option is kept only for completeness; it is **not recommended** for automated use.

## Troubleshooting

### “Access denied” or patching fails
- Make sure you run PowerShell **as Administrator** (the script tries to self‑elevate, but some policies may block it).  
- Close the game you want to patch - the file cannot be locked.

### `patchlaa` crashes on some files
- Some old or corrupted executables may cause `patchlaa` to panic. This is rare (about 2% of files in my tests). Use `editbin` for those specific files if needed.

### 4GB Patch opens file‑picker dialog
- That is a known bug in the CLI mode of that tool, not an issue with the script. Abort the script and choose `patchlaa` or `editbin` instead.

### `editbin` not found
- Install Visual Studio Build Tools (free) and re‑run the script, or provide the full path when prompted.

### “Execution Policy” error
- Run this command first to allow script execution for the current session:
  ```powershell
  Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

## Credits
- patchlaa by [slonkazoid](https://github.com/slonkazoid)
- 4GB Patch by [NTCore](https://ntcore.com)
- editbin from [Microsoft Visual Studio](https://visualstudio.microsoft.com)

*Script vibe coded by [idontknowshark](https://github.com/idontknowshark), generated by DeepSeek.*
 - *visit https://bordiga.party if you like bordiga*
 - *visit https://rateyourmusic.com/~idontknowshark if you like black metal*
