# Reference files — Steris lab

Read-only copies from the **Curia ABQ Steris simulation lab** for use while reading lessons and doing labs. You do not import these into a running project from here — they are for **study** (grep, Studio offline open, diff, ActiveX blob inspection).

Canonical copies also live in [Curia_ABQ](https://github.com/JGSum/Curia_ABQ) (`ACDs/`, `DisplaysXMLs/`). Refresh this folder when the lab PLCs or displays change materially.

## PLC (`plc/`)

| File | Unit | Echo IP | Use in lessons |
|------|------|---------|----------------|
| `BX45183A_lab.L5X` | Decon autoclave (5380) | `192.168.100.70` | Lesson 6 — alarms; Lesson 7 — `Print_main`, `ReadyToPrint`, `LastCycle[]` |
| `GMP_Lab_lab.L5X` | Parts Washer (5069-L320ER) | `192.168.100.100` | Lesson 7 — `asPrintBuffer[]`, `FC60_PRINT`, `FC66_PRNT_CYC_HD`, `bHMIDataToPrint` |

Open in Studio 5000 offline or search with ripgrep / VS Code.

## HMI displays (`hmi/`)

FT View SE **display XML exports** (same content family as `.gfx` in the project). Names match displays in `4401_HMI_Server` / `ABQ4401FTV`.

| File | Role | Lessons / labs |
|------|------|----------------|
| `steris_decon_electronic_signature.xml` | Decon cycle report e-sign, **ListBox1** (FM20) | Labs 4, 5, 7 |
| `steris_decon_print_control.xml` | Hidden display — appends `Reprint.txt`, `ReadyToPrint` handshake | Lab 7 |
| `steris_decon_header.xml` | Decon startup / login / header | Lab setup context |
| `steris_sterility_electronic_signature.xml` | Sterility e-sign (same ActiveX pattern as Decon) | Lab 4 |
| `HEADER.xml` | PW header — `PrintFile`, VersaView print handshake | Lesson 7 |
| `80 - PRINT MENU.xml` | PW print menu entry | Lesson 4 |
| `81 - PRINT VIEW.xml` | PW report viewer ListBox | Labs 4, 7 |
| `40 - CYCLE RUN MENU.xml` | PW cycle run — `tPrintingCompleted` wait state | Lesson 4 |

### ActiveX version hints

Search display XML for `FM20.DLL` and version strings when debugging “newer version required” (Labs 4–5).

### VBA source

Display VBA logic is also exported as `.cls` in `Curia_ABQ/Code/` (`Decon.cls`, `Header.cls`, etc.) — see `Curia_ABQ/docs/ftview-vba-exports.md`.
