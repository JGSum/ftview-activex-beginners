# Lab 5 — SysWOW64 FM20 fix (real lab scenario)

**Goal:** Reproduce and fix *“Newer file version required”* when CAB + IIS are already correct.

**Where:** Lab VM with FactoryTalk View Studio (32-bit).

**Files:** Use `Curia_ABQ/Lab_Back_up/FM20DLL.CAB` (copy to IIS path per [Lab 2](lab-02-map-the-deployment-path.md)).

## Symptoms

- CAB downloads (Diagnostics: *“CAB file downloading from server…”*)
- Still errors: *“Newer file version required”* + *“Problem setting up ActiveX control(s)”*
- Extracted CAB shows `FM20.DLL` **16.0.10357.20081**

## Diagnose (cmd)

```powershell
Get-Item "$env:windir\SysWOW64\FM20.DLL" | Select-Object FullName, @{N='FileVersion';E={$_.VersionInfo.FileVersion}}
```

If **FileVersion** is older than display XML (e.g. `11.0.8000` vs `16.0.10357.20081`), SysWOW64 is the problem.

## Fix (manual)

1. Close View Studio.
2. `C:\Windows\SysWOW64\` → rename `FM20.DLL` → `FM20.DLL.bak`
3. Copy `FM20.dll` from CAB extract (`%TEMP%\fm20-check\` or extract fresh) into `SysWOW64\`
4. Re-run version check (cmd above).
5. Open `steris_decon_electronic_signature` (or e-sign display) in Studio.

## Fix (cmd alternative)

After extract to `%TEMP%\fm20-check\`:

```powershell
Stop-Process -Name "ViewStudio" -ErrorAction SilentlyContinue
Rename-Item "$env:windir\SysWOW64\FM20.DLL" FM20.DLL.bak -Force -ErrorAction SilentlyContinue
Copy-Item "$env:TEMP\fm20-check\FM20.dll" "$env:windir\SysWOW64\FM20.DLL" -Force
Get-Item "$env:windir\SysWOW64\FM20.DLL" | Select-Object @{N='FileVersion';E={$_.VersionInfo.FileVersion}}
```

## Success criteria

- [ ] SysWOW64 `FM20.DLL` version matches display requirement
- [ ] E-sign / Print View display opens without ActiveX error

## Reflection

Why didn’t copying the CAB to inetpub alone fix Studio?
