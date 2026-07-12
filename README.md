# FactoryTalk View, ActiveX & PLC — Beginner Lessons

Hands-on lessons from industrial HMI and PLC work — **ActiveX** / ListBox / CAB deployment, plus **Logix alarm FOR-loop** indirection.

Built from real troubleshooting (Steris cycle reports, PDF printers, lab VM setup, Decon alarm tracing).

**Lab environment:** Use the restored **Curia ABQ** simulation kit: [Lab_Back_up](https://github.com/JGSum/Curia_ABQ/tree/main/Lab_Back_up) — ACDs, HMI backup, PDF24 MSI, FM20 CAB, `pdf_check_v1.8.exe`, and step-by-step VM setup (`LAB_SETUP_INSTRUCTIONS.txt`). Labs 1–7 are written against that machine.

## Who this is for

- You use FactoryTalk View SE but ActiveX errors feel like magic
- You want to understand **CAB files**, **ListBox**, and **why Studio says FM20DLL.CAB is missing**
- You prefer **manual steps** with optional PowerShell shortcuts

## How to use

1. Read lessons in order (`lessons/`).
2. Do labs on a **non-production** VM or your lab machine (`labs/`).
3. Optional: use the same production HMI RDP read-only checks from Lab 2–3.

## Lessons

| # | Topic |
|---|--------|
| [01](lessons/01-what-is-activex.md) | What ActiveX is (and is not) |
| [02](lessons/02-cab-files.md) | CAB archives — extract, copy, deploy |
| [03](lessons/03-activex-in-factorytalk-view.md) | How View uses ActiveX on displays |
| [04](lessons/04-listbox-and-cycle-reports.md) | ListBox vs PDF printer — two different jobs |
| [05](lessons/05-what-are-dlls.md) | What DLLs are — CAB vs SysWOW64, today’s lab fix |
| [06](lessons/06-plc-alarm-for-loop-indirection.md) | PLC alarm FOR loop — `Alarm_ACT` vs `Alarm_ON`, `[DINT_IX].[BIT_IX]` |
| [07](lessons/07-plc-cycle-report-print-pipeline.md) | PLC cycle report & print — `Print_main` / `asPrintBuffer` → HMI → PDF24 |

## Labs

| # | Topic |
|---|--------|
| [Lab 1](labs/lab-01-extract-and-inspect-cab.md) | Extract `FM20DLL.CAB` and find `FM20.DLL` |
| [Lab 2](labs/lab-02-map-the-deployment-path.md) | Folder path ↔ HTTP URL ↔ Studio error |
| [Lab 3](labs/lab-03-printer-name-vs-driver.md) | `Get-Printer` — queue name vs driver (PDFCreator/PDF24) |
| [Lab 4](labs/lab-04-open-a-listbox-display.md) | Open an e-sign or Print View display in Studio |
| [Lab 5](labs/lab-05-syswow64-fm20.md) | Fix “newer version required” — replace SysWOW64 FM20.DLL |
| [Lab 6](labs/lab-06-build-your-own-alarm-handler.md) | Build a 3-alarm FOR-loop handler in Studio (Decon pattern) |
| [Lab 7](labs/lab-07-trace-plc-print-pipeline.md) | Trace Decon/PW print handshake on Echo lab PLCs |

## Related real-world docs

**Lab kit:** [Curia_ABQ/Lab_Back_up](https://github.com/JGSum/Curia_ABQ/tree/main/Lab_Back_up) — restore VMs from scratch, then run Labs 1–7.

**Session docs:** `Curia_ABQ/docs/lab-setup-plan.md`, `docs/lab-decon-cycle-stepthrough.md`, `docs/lab-cycle-simulation-and-report-process.md`, `docs/plc-changes-lab-only.md`.

## License

MIT — use and adapt for training.
