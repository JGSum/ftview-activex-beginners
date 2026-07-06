# Lesson 2 — CAB files

## What is a CAB?

**CAB** = Microsoft **cabinet** archive (compressed bundle). Think zip, but Microsoft-native.

Rockwell puts ActiveX DLLs in CABs so clients can **download and register** them automatically.

## Typical production path

```text
C:\Users\Public\Documents\RSView Enterprise\ActiveX Control Setup\FM20Dll.Cab
```

(URL path is related but not identical:)

```text
http://<HMI-PC-NAME>/RSViewActiveXControlSetup/FM20DLL.CAB
```

## What you normally DO with a CAB

| Task | When |
|------|------|
| **Copy** from working machine to lab | “CAB file missing” |
| **Extract** to inspect contents | Learning / verifying version |
| **Replace** with production copy | Version mismatch errors |

## What you normally do NOT do

- Open in Notepad (binary gibberish)
- Hand-build a CAB unless you know `makecab` packaging rules
- Assume one CAB fixes every ActiveX on the project (each control can have its own CAB)

## Extract manually (Windows)

1. Right-click `FM20Dll.Cab`
2. **Extract All…** (or use 7-Zip)
3. Look for **`FM20.DLL`**

## Extract with PowerShell (optional)

```powershell
expand.exe -F:* "C:\path\to\FM20Dll.Cab" "C:\temp\fm20-extract"
```

## Version matters

Display XML may record:

```text
fileVersion="16.0.10357.20081"
```

If Studio says **“Newer file version required”**, your CAB/DLL is **older** than the display expects — copy from a machine where that display already works.

## Check yourself

1. What file inside the CAB does ListBox need?
2. Why copy from production instead of downloading random FM20 from the internet?

<details>
<summary>Answers</summary>

1. `FM20.DLL` (Microsoft Forms 2.0)
2. Version must match what the **display project** was built with; wrong DLL can fail silently or break registration.

</details>

**Next:** [Lesson 3 — ActiveX in FactoryTalk View](03-activex-in-factorytalk-view.md)
