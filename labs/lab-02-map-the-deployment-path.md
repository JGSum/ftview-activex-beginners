# Lab 2 — Map folder path to HTTP URL

**Goal:** Understand why Studio errors show a **URL** when you already copied a CAB to `C:\...`.

## Background

Studio may say:

```text
http://WIN-K76QMLETJEL/RSViewActiveXControlSetup/FM20DLL.CAB
```

The file on disk and the URL must both work for some clients.

## Manual path

1. Confirm CAB exists:
   `C:\Users\Public\Documents\RSView Enterprise\ActiveX Control Setup\FM20Dll.Cab`
2. On the **same PC**, open a browser (Edge/IE)
3. Try: `http://localhost/RSViewActiveXControlSetup/FM20DLL.CAB`
4. Also try: `http://127.0.0.1/RSViewActiveXControlSetup/FM20DLL.CAB`
5. Note: download prompt / file opens = good; 404 = web share not mapped

### If 404 — common disk locations to search (manual)

Search File Explorer for folder name **`RSViewActiveXControlSetup`** (not the CAB).

Typical Rockwell/IIS paths (one may exist on your box):

- `C:\inetpub\wwwroot\RSViewActiveXControlSetup\`
- Under `C:\Program Files (x86)\Rockwell Software\RSView Enterprise\`

If you find that folder, copy the same CAB there.

## Optional command path

```powershell
Test-Path "$env:PUBLIC\Documents\RSView Enterprise\ActiveX Control Setup\FM20Dll.Cab"
Invoke-WebRequest -Uri "http://localhost/RSViewActiveXControlSetup/FM20DLL.CAB" -Method Head -UseBasicParsing
```

(Head request = check exists without downloading whole file.)

## Success criteria

- [ ] CAB on disk: yes/no
- [ ] HTTP URL responds: yes/no
- [ ] If only disk works, you know web deploy still needs fixing
