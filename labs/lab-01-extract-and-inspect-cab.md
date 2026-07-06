# Lab 1 — Extract and inspect a CAB

**Goal:** See what’s inside `FM20DLL.CAB` and connect it to ListBox.

**Where:** Lab VM or production HMI (read-only copy is fine).

## Manual path

1. Open File Explorer to the CAB folder:
   `C:\Users\Public\Documents\RSView Enterprise\ActiveX Control Setup\`
2. Right-click **`FM20Dll.Cab`** → **Extract All…** → choose `C:\temp\fm20-lab`
3. Open the extract folder — confirm **`FM20.DLL`** exists
4. Right-click `FM20.DLL` → **Properties** → **Details** tab — note **File version**

## Optional command path

```powershell
New-Item -ItemType Directory -Force -Path C:\temp\fm20-lab | Out-Null
expand.exe -F:* "$env:PUBLIC\Documents\RSView Enterprise\ActiveX Control Setup\FM20Dll.Cab" C:\temp\fm20-lab
Get-Item C:\temp\fm20-lab\FM20.DLL | Select-Object Name, Length, VersionInfo
```

## Success criteria

- [ ] You found `FM20.DLL` inside the CAB
- [ ] You can state the file version in one line

## Reflection

Why is the DLL useless in Notepad but essential in View?
