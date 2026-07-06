# Lesson 3 — ActiveX in FactoryTalk View

## Two moments ActiveX matters

### 1. Design time (View Studio)

Engineers open displays on a dev PC. Studio must load ActiveX to show the real control (not just a hatched placeholder).

### 2. Run time (View SE Client)

Operators open the same display. The **client PC** loads/registers the control — often via HTTP from the HMI server name in the error URL.

## The error message decoded

```text
Unable to load ActiveX control for display steris_sterility_electronic_signature.
cause: Microsoft Forms 2.0 ListBox: CAB file missing on the server:
http://WIN-K76QMLETJEL/RSViewActiveXControlSetup/FM20DLL.CAB
```

| Piece | Meaning |
|-------|---------|
| Display name | Which `.gfx` failed |
| Microsoft Forms 2.0 ListBox | Which ActiveX type |
| `WIN-K76QMLETJEL` | PC name clients ask for the CAB |
| `RSViewActiveXControlSetup` | IIS/web folder alias |
| `FM20DLL.CAB` | Package to download |

## Folder vs URL

Copying CAB to the **Rockwell folder** is step one. If the URL still 404s, IIS (or View’s web service) may not expose `RSViewActiveXControlSetup` — that’s a separate fix (Lab 2).

## exposeToVba in display XML

ListBox on e-sign often has:

```text
exposeToVba="vbaControl"
```

That lets VBA do `ListBox1.AddItem ...` to fill report lines from `C:\Data\Reprint.txt`.

Controls with `notExposed` are decorative overlays — VBA can’t touch them.

## Fixing strategy (order)

1. Copy **known-good CAB** from production → lab folder
2. Retry opening display in Studio
3. If “newer version” → CAB from machine where display works
4. If “CAB missing” but file exists → check **HTTP path** (Lab 2)
5. Register control via View **Tools → ActiveX Control Setup** if needed (version-dependent)

## Check yourself

Does fixing ActiveX also install PDF24?

<details>
<summary>Answer</summary>

No. ActiveX = on-screen ListBox. PDF = separate Windows printer install.

</details>

**Next:** [Lesson 4 — ListBox vs PDF](04-listbox-and-cycle-reports.md)
