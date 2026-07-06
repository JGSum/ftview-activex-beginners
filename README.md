# FactoryTalk View & ActiveX — Beginner Lessons

A short, hands-on introduction to **ActiveX controls** in industrial HMI work — especially Microsoft Forms ListBox (`FM20.DLL`) and Rockwell **CAB** deployment.

Built from real troubleshooting (Steris cycle reports, PDF printers, lab VM setup).

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

## Labs

| # | Topic |
|---|--------|
| [Lab 1](labs/lab-01-extract-and-inspect-cab.md) | Extract `FM20DLL.CAB` and find `FM20.DLL` |
| [Lab 2](labs/lab-02-map-the-deployment-path.md) | Folder path ↔ HTTP URL ↔ Studio error |
| [Lab 3](labs/lab-03-printer-name-vs-driver.md) | `Get-Printer` — queue name vs driver (PDFCreator/PDF24) |
| [Lab 4](labs/lab-04-open-a-listbox-display.md) | Open an e-sign or Print View display in Studio |
| [Lab 5](labs/lab-05-syswow64-fm20.md) | Fix “newer version required” — replace SysWOW64 FM20.DLL |

## Related real-world docs

If you work on Curia ABQ Steris lab: see `Curia_ABQ/docs/lab-setup-plan.md` and `docs/autoclave-pdf-flow-production.md`.

## License

MIT — use and adapt for training.
