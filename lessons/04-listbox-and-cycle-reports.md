# Lesson 4 — ListBox vs PDF on cycle reports

## Autoclave flow (working production)

```text
PLC strings → C:\Data\Reprint.txt
     ↓
E-sign display opens
     ↓
ListBox1 shows report lines  ← ActiveX (FM20)
     ↓
Operator Accept → FT View Electronic Signature dialog
     ↓
VBA PrintLastCycle → C:\Data\Reports\AC_Batch_....txt
     ↓
NOTEPAD /PT file "PDFCreator"  ← virtual printer (PDF24 driver)
     ↓
PDF24 Assistant → Save As PDF
```

## Two different Windows integrations

| Piece | Technology | Job |
|-------|------------|-----|
| Report viewer | ActiveX ListBox + VBA | Show text, operator review |
| PDF file | Virtual printer + Notepad | Create PDF file |

## Printer name trap (real production example)

```text
Get-Printer → Name: PDFCreator, DriverName: PDF24
```

VBA uses the **queue name** `PDFCreator`, not the word `PDF24`. Lab must install PDF24 but keep (or rename to) that queue name.

## Parts Washer difference

PW often stays on **`40 - CYCLE RUN MENU`** with “PRINTING NOT COMPLETETED” — that’s a **PLC tag** (`tPrintingCompleted`), not ActiveX.

PW PDF path is more manual (`80`/`81` Print Menu) or via `HEADER.PrintFile` — still uses a virtual printer string, separate from ListBox.

## Check yourself

Operator says “PDF builder never opened” on PW during cycle run. Is that likely an ActiveX problem?

<details>
<summary>Answer</summary>

Not necessarily. PW may not auto-open PDF at cycle end at all; check printer install, report master, and print-complete tags before blaming ListBox.

</details>

**Labs:** [Lab 5 — SysWOW64 FM20](../labs/lab-05-syswow64-fm20.md) · [Lesson 5 — DLLs](../lessons/05-what-are-dlls.md)
