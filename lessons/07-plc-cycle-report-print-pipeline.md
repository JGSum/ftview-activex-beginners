# Lesson 7 — PLC cycle report and print pipeline (HMI → PLC → PDF24)

**From real lab:** Curia ABQ Steris simulation — `BX45183A_lab.L5X` (Decon) and `GMP_Lab_lab.L5X` (Parts Washer) on FactoryTalk Logix Echo. L5X exports: `Curia_ABQ/ACDs/`.

This lesson is **PLC-heavy**. The HMI and PDF24 steps exist, but the report **content** and **timing** are decided in Logix.

---

## The big picture (two different PLC designs)

Both units end at a `.txt` file and a virtual printer, but the PLC architectures differ:

| | **Decon / autoclave** (`BX45183A_lab`) | **Parts Washer** (`GMP_Lab_lab`) |
|---|----------------------------------------|----------------------------------|
| Report storage | `LastCycle[2500]` circular STRING buffer | `asPrintBuffer[600]` STRING array |
| Line-at-a-time to HMI | `Data_print` (one STRING per handshake) | `asPrintBuffer[0]` + `bHMIDataToPrint` |
| HMI trigger | `ReadyToPrint` pulse | `bHMIDataToPrint` BOOL |
| HMI done | `Printing.PrintingDone` → `PrintingDone` | `bHMIPrintDone` |
| Main print program | `Print_main` | `FC60_PRINT` + FC66/67/68… helpers |
| End-of-cycle gate | `E_Sign_Screen` at step 26/30 | PW: manual / HEADER path (different) |

Same operator story at the end — review report, Accept, PDF — but **learn the PLC side first**.

---

## End-to-end flow (Decon — best lab path)

```text
[PLC during cycle]
  Print_main runs every scan
    → JSR P1…P14 subroutines append lines to LastCycle[S_Pointer]
    → Data_save ON while cycle runs

[PLC → HMI live append]
  P_comm state machine reaches command 6
    → COP LastCycle[P_Pointer] → Data_print
    → OTL ReadyToPrint
  HMI print_control VBA sees ReadyToPrint
    → appends Data_print line to C:\Data\Reprint.txt
    → sets Printing.PrintingDone
  PLC sees PrintingDone → clears ReadyToPrint, advances P_Pointer

[PLC end of cycle]
  Step 26 or 30 + door closed → OTL E_Sign_Screen
  Process still running; Data_save stays on until e-sign resolved
  Rung: E_Sign_Screen + Printing_cmd_inactive → OTU Data_save (freeze buffer)

[HMI end — thin layer]
  steris_decon_electronic_signature opens; ListBox shows Reprint.txt
  Accept → PrintLastCycle VBA → DC_Batch_*.txt → NOTEPAD /PT → PDFCreator
  pdf_check.exe (optional) → C:\Data\PDF\*.pdf
```

---

## Decon PLC deep dive — `Print_main`

Open **`Print_main`** in `BX45183A_lab.L5X` (program **Print**).

### 1. `Data_save` — “recording is on”

When the cycle starts (`Process.Cycle_start`), **`OTL(Data_save)`** turns on. While `Data_save` is true, measurement/phase/alarm subroutines **COP** formatted strings into **`LastCycle[S_Pointer]`** and increment `S_Pointer`.

At end of cycle (Rung 25):

```
XIC(Process.Running) XIC(E_Sign_Screen) … XIC(Printing_cmd_inactive)
  → OTU(Data_save)
  → MOVE(ROW_NR, PRT_ROW)
```

**Meaning:** PLC stops appending new lines when the e-sign display opens and printing commands have gone idle. The report snapshot is frozen.

### 2. `P_comm` — print command state machine

`P_comm` selects which print job runs (parameters, phase, measurements, alarms, cycle data, PID, etc.). Examples from `Print_main`:

| P_comm | JSR / action |
|--------|----------------|
| 1 | `P1_Precycle` — cycle parameters at start |
| 3 | `P3_Measurements` — on step change or interval timer |
| 4 | `P4_Alarm` — alarm lines during cycle |
| 6 | `P6_Cycle_data` — **replay LastCycle to HMI** |

Command 6 is the HMI streaming path.

### 3. `LastCycle[]` vs `P_Pointer` / `S_Pointer`

- **`S_Pointer`** — write index while **building** the report during the cycle.
- **`P_Pointer`** — read index while **playing back** to the HMI.
- When `P_Pointer < S_Pointer`, there are lines waiting to send.

Rung 14 (command 6 start):

```
EQ(P_comm,0) LT(P_Pointer,S_Pointer)
  → MOVE(6,P_comm)
  → COP(LastCycle[P_Pointer], Data_print, 1)
  → ADD(P_Pointer,1,P_Pointer)   (when not waiting on ReadyToPrint)
```

Each loop places **one line** in `Data_print`.

### 4. `ReadyToPrint` / `PrintingDone` handshake

In **`P6_Cycle_data`** (or adjacent print comm logic):

```
EQ(P_comm,6) … TON(Print_delay) … ONS(readytoprint_oneshot) OTL(ReadyToPrint)
XIC(PrintingDone) → OTU(ReadyToPrint), CLR(P_comm), …
```

**PLC waits** until HMI sets **`PrintingDone`** before sending the next line. That is why `print_control` must be running in the FT View client — without it, `ReadyToPrint` sticks high and the handshake stalls.

Rung 13 in `Print_main`:

```
XIO(PrintingACT) JMP(jump)
```

If **`PrintingACT`** is false (HMI not ready), the whole print main body is skipped.

### 5. `E_Sign_Screen` and `HMI.Force_NS`

At step 26/30, when the door is closed:

```
OTL(E_Sign_Screen)
MOVE(100, HMI.Force_NS)
OTL(HMI.Force_On_NS)
```

**100** is the PLC command code that tells the HMI to open the **cycle report / e-sign** display. This is not VBA magic — the sequencer sets a force word the HMI obeys.

---

## Parts Washer PLC deep dive — `asPrintBuffer[]`

Open **`GMP_Lab_lab.L5X`**. Search tags: **`asPrintBuffer`**, **`dPrintArrayPtr`**, **`bHMIDataToPrint`**, **`bHMIPrintDone`**, **`bVersaView`**.

### 1. Building lines — the CONCAT pattern

Most sequence FC routines append one report line like this:

```
CONCAT(label_string, value_string, asPrintBuffer[dPrintArrayPtr])
CONCAT(asPrintBuffer[dPrintArrayPtr], sCrLf, asPrintBuffer[dPrintArrayPtr])
ADD(dPrintArrayPtr, 1, dPrintArrayPtr)
```

- **`asStringPrintTreatParameters1[]`** — label templates (“DATE: ”, “CYCLE NUMBER: ”, …)
- **`asConcatenate[]`** — date/time substrings built in **`FC60_PRINT`**
- **`dPrintArrayPtr`** — next free row in the buffer (0..599)
- **`ONS(...)`** on sequence bits — print **once per event**, not every scan

Example from FC03 (cycle start header):

```
XIC(abSeqRun[2]) ONS(...)
  → OTL(bReqPrintHeader)
  → JSR(FC66_PRNT_CYC_HD,0)
```

### 2. `FC60_PRINT` — date/time string factory

**FC60_PRINT** runs on a schedule and builds **`asConcatenate[4]`** (date) and **`asConcatenate[9]`** (time) from **`adDateFormatFromPLC[]`** and **`iDateFormatFromHMI`**.

Rung 2 (date) — order depends on `iDateFormatFromHMI`:

- `0` → DD-MM-YYYY style inserts
- `1` → MM-DD-YYYY
- **`2` → YYYY-MM-DD** (lab/production target)

Separator character comes from **`asConcatenate[0]`** (default `'-'`; lab change to `'/'` for `YYYY/MM/DD`).

Rung 4 builds **`sAlarmHeader`** = date + time prefix used on alarm lines.

Subroutines **`FC67_PRNT_CYC_FT`**, **`FC68_PRNT_PRCSS_VL`**, etc. add more sections when enabled.

### 3. `FC66_PRNT_CYC_HD` — cycle header block

When **`bReqPrintHeader`** is true:

| Rung | Line content |
|------|----------------|
| 1 | `CYCLE: …` from `asStringToHMI[50]` |
| 2 | **`DATE: …`** from `asConcatenate[4]` |
| 3 | **`START: …`** from `asConcatenate[9]` |
| 4 | `CYCLE NUMBER: …` from HMI value |
| 5+ | Unit number, batch ID, etc. |

Each rung uses the same **`asPrintBuffer[dPrintArrayPtr]` + ADD** pattern.

### 4. HMI handshake — `bVersaView` path (lab FT View)

In **`FC55`** (search `bHMIDataToPrint`):

```
XIC(bVersaView) XIO(bHMIPrintDone) GT(dPrintArrayPtr,0) XIO(bResetPrinter)
  → OTE(bHMIDataToPrint)
```

When there are buffered lines and HMI has not acknowledged:

- PLC raises **`bHMIDataToPrint`**
- HMI HEADER display reads **`asPrintBuffer[0]`**, writes the line to disk, clears index 0, shifts array
- HMI sets **`bHMIPrintDone`** pulse
- PLC sees done, decrements **`dPrintArrayPtr`**, sends next line

Alternate path: **`udtHardwareConfig.bPrinter`** sends **`Local:1:O0.ASCII.*`** directly to a serial/ASCII card (production printer hardware) — same buffer, different sink.

### 5. Alarms on the report — `FC41`

Alarm print rungs mirror the alarm FOR-loop idea from Lesson 6, but output goes to the report:

```
XIC(adAlarmFlag[1].22) ONS(...)
  → CONCAT(sAlarmHeader, …, asPrintBuffer[dPrintArrayPtr])
  → CONCAT(asAlarmMessages[55], sCrLf, asPrintBuffer[dPrintArrayPtr])
  → ADD(dPrintArrayPtr, 1, dPrintArrayPtr)
```

Each alarm number gets its own ONS rung — **direct index**, not `[DINT_IX].[BIT_IX]`, because these are one-shot print events, not a scanning latch loop.

---

## HMI layer (short — already Lesson 4)

| Step | Decon | PW |
|------|-------|-----|
| Live file | `print_control` → `Reprint.txt` | HEADER → `PrintBuffer.txt` / cycle report file |
| Review | e-sign ListBox (ActiveX) | `81 - PRINT VIEW` or cycle-end wait |
| Finalize | `PrintLastCycle` VBA | `PrintFile` / manual print menu |
| PDF | `NOTEPAD /PT … "PDFCreator"` | Same queue name |
| Batch sweep | `pdf_check_v1.8.exe` | Same |

Queue name **`PDFCreator`** with **PDF24** driver — see [Lab 3](../labs/lab-03-printer-name-vs-driver.md).

---

## What to watch in Studio (either PLC)

| Tag | Decon meaning | PW meaning |
|-----|---------------|------------|
| `Data_save` / `dPrintArrayPtr` | Recording on / lines queued | Lines in buffer |
| `P_Pointer` vs `S_Pointer` | Playback vs write index | — |
| `ReadyToPrint` / `bHMIDataToPrint` | Line ready for HMI | Line ready for HMI |
| `PrintingDone` / `bHMIPrintDone` | HMI consumed line | HMI consumed line |
| `E_Sign_Screen` | Open report display | — |
| `HMI.Force_NS` | HMI command word (=100 e-sign) | PW equivalents differ |

---

## Check yourself

During a Decon lab cycle, `ReadyToPrint` toggles but `Reprint.txt` stays empty. Which layer failed first?

<details>
<summary>Answer</summary>

HMI **`print_control`** display not running in the FT View **client** (not Studio preview). PLC is doing its job; the handshake has no consumer. Confirm startup macro loads hidden displays.

</details>

PW: `dPrintArrayPtr` counts up during the cycle but `bHMIDataToPrint` never latches. What tag must be true for the VersaView path?

<details>
<summary>Answer</summary>

**`bVersaView`** must be 1 (FT View / VersaView print mode). Also need `dPrintArrayPtr > 0`, `bHMIPrintDone` false, and HEADER running to respond.

</details>

---

**Lab:** [Lab 7 — Trace the PLC print pipeline on Echo](../labs/lab-07-trace-plc-print-pipeline.md)

**Related:** [Lesson 4 — ListBox vs PDF](04-listbox-and-cycle-reports.md) · [Lesson 6 — Alarm indirection](06-plc-alarm-for-loop-indirection.md) · `Curia_ABQ/docs/lab-cycle-simulation-and-report-process.md`
