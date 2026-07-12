# Lab 7 — Trace the PLC print pipeline on Echo

**Goal:** Follow one report line from PLC logic to the HMI handshake, using the **Curia ABQ lab** already restored from `Lab_Back_up/`.

**Time:** ~60–90 min (Decon path required; PW path optional second half).

**Reference:** [Lesson 7 — PLC cycle report print pipeline](../lessons/07-plc-cycle-report-print-pipeline.md)

**Prerequisites:** Lab built per `Curia_ABQ/Lab_Back_up/LAB_SETUP_INSTRUCTIONS.txt` — Echo PLCs at `.70` (Decon) and `.100` (PW), FT View client with hidden print displays.

---

## Part A — Restore context (5 min)

Confirm you are on the **lab VM**, not production.

| Item | Expected |
|------|----------|
| Decon PLC | `BX45183A_lab` @ `192.168.100.70` |
| PW PLC | `GMP_Lab_lab` @ `192.168.100.100` |
| L5X exports (reference) | `Curia_ABQ/ACDs/BX45183A_lab.L5X`, `GMP_Lab_lab.L5X` |
| HMI | `ABQ4401FTV.apb` restored — Decon + PW in one project |

---

## Part B — Decon: map `Print_main` (20 min)

1. Studio 5000 → open **`BX45183A_lab`** (online to Echo `.70` or offline with L5X).
2. Program **Print** → routine **`Print_main`**.
3. Find and bookmark these rungs (use Comments):

| Rung topic | Tags to note |
|------------|--------------|
| `Data_save` ON at cycle start | `Process.Cycle_start`, `Data_save` |
| Command 6 / cycle data replay | `P_comm`, `P_Pointer`, `S_Pointer`, `Data_print` |
| `XIO(PrintingACT) JMP` | `PrintingACT` |
| End cycle freeze | `E_Sign_Screen`, `Data_save`, `Printing_cmd_inactive` |

4. Open **`P6_Cycle_data`** — find `ReadyToPrint` and `PrintingDone` logic.

**Write down:** What condition must be true for `COP(LastCycle[P_Pointer], Data_print, 1)` to run?

<details>
<summary>Answer</summary>

`P_comm = 6`, `P_Pointer < S_Pointer`, and not waiting on `ReadyToPrint` / `Print_delay` for the previous line.

</details>

---

## Part C — Decon: live tag monitor during cycle (30 min)

1. Launch **Steris_Decon** FT View **client** (not Studio). Confirm startup loads **`print_control`**.
2. Studio → tag monitor — add:

```
Data_save
P_Pointer
S_Pointer
P_comm
Data_print
ReadyToPrint
PrintingDone
PrintingACT
E_Sign_Screen
Process.Step
```

3. Run a **short lab cycle** (or simulate to step 4+). Do **not** force print tags.

4. During the cycle, watch:

| Observation | Pass |
|-------------|------|
| `Data_save` latches ON after start | ☐ |
| `S_Pointer` increments over time | ☐ |
| `P_comm` becomes 6 at some point | ☐ |
| `ReadyToPrint` pulses | ☐ |
| `P_Pointer` increments after each pulse | ☐ |
| `C:\Data\Reprint.txt` grows | ☐ |

5. At cycle end (step 26/30):

| Observation | Pass |
|-------------|------|
| `E_Sign_Screen` = 1 | ☐ |
| `Data_save` drops to 0 | ☐ |
| E-sign display opens on HMI | ☐ |

6. Accept → PDF path (Lesson 4) — confirm `DC_Batch_*.txt` in `C:\Data\Reports\`.

**If `ReadyToPrint` sticks ON and `P_Pointer` stops:** HMI handshake broken — check `print_control` and `Printing.PrintingDone` mapping.

---

## Part D — PW: `asPrintBuffer` and FC60/FC66 (20 min)

1. Open **`GMP_Lab_lab`** (Echo `.100` or L5X).
2. Search controller tags: **`asPrintBuffer`**, **`dPrintArrayPtr`**, **`bHMIDataToPrint`**, **`bHMIPrintDone`**, **`bVersaView`**.
3. Open routines:

| Routine | What to find |
|---------|----------------|
| **`FC60_PRINT`** | Rung 2 — `asConcatenate[4]` date build; `iDateFormatFromHMI`, `asConcatenate[0]` separator |
| **`FC66_PRNT_CYC_HD`** | Rung 2 — `DATE:` line into `asPrintBuffer[dPrintArrayPtr]` |
| **`FC55`** (or search `bHMIDataToPrint`) | VersaView handshake rung |

4. Pick **one** sequence FC (e.g. FC03) and trace a single CONCAT rung:

```
CONCAT(label, value, asPrintBuffer[dPrintArrayPtr])
ADD(dPrintArrayPtr, 1, dPrintArrayPtr)
```

**Write down:** What ONS bit prevents that line from printing every scan?

---

## Part E — PW: optional live monitor

Add to tag monitor:

```
dPrintArrayPtr
bHMIDataToPrint
bHMIPrintDone
bVersaView
asPrintBuffer[0]
```

Run PW cycle or partial sequence. Pass if `dPrintArrayPtr` rises during events and `bHMIDataToPrint` toggles when `bVersaView = 1`.

---

## Part F — Compare designs (5 min)

Fill in from memory:

| Question | Decon | PW |
|----------|-------|-----|
| Main buffer tag | | |
| Index tags | | |
| HMI “line ready” tag | | |
| HMI “done” tag | | |
| Date line built in | | |

<details>
<summary>Answers</summary>

| Question | Decon | PW |
|----------|-------|-----|
| Main buffer | `LastCycle[]` | `asPrintBuffer[]` |
| Index | `P_Pointer` / `S_Pointer` | `dPrintArrayPtr` |
| Line ready | `ReadyToPrint` | `bHMIDataToPrint` |
| Done | `PrintingDone` | `bHMIPrintDone` |
| Date line | Built in `Print_time` / header JSRs | **`FC60_PRINT`** → **`FC66_PRNT_CYC_HD`** rung 2 |

</details>

---

## Success criteria

- [ ] Can explain **`Data_save`** vs **`E_Sign_Screen`** on Decon without opening the ladder
- [ ] Saw **`ReadyToPrint`** pulse at least once during a lab cycle
- [ ] Can describe the **`CONCAT` + `dPrintArrayPtr`** pattern on PW
- [ ] Know where **`DATE:`** is formatted in PLC (`FC66` + `FC60`, not the HMI clock)

---

## Related labs

- [Lab 3 — Printer name vs driver](lab-03-printer-name-vs-driver.md) — PDF24 / `PDFCreator`
- [Lab 4 — Open ListBox display](lab-04-open-a-listbox-display.md) — e-sign viewer
- [Lab 6 — Alarm FOR loop](lab-06-build-your-own-alarm-handler.md) — same Decon PLC, different subroutine
