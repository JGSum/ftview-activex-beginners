# Lab 3 — Printer queue name vs driver

**Goal:** Learn why VBA says `PDFCreator` while the UI says PDF24.

**On the Steris lab VM:** PDF24 should be installed with the printer queue renamed to **`PDFCreator`** (see `Curia_ABQ/docs/lab-setup-plan.md`).

## Manual path

1. **Settings → Bluetooth & devices → Printers & scanners**
2. List every printer with **PDF** in the name
3. Click **PDFCreator** → **Printer properties**
4. Record **Model** and **Comment** fields

## Optional command path

```powershell
Get-Printer | Where-Object { $_.Name -match 'PDF' } | Format-Table Name, DriverName -AutoSize
```

## Exercise

Production example:

| Name | DriverName |
|------|------------|
| PDFCreator | PDF24 |

VBA line:

```vb
xlApp.Run ("notepad /PT """ & filepath & """ ""PDFCreator""")
```

**Question:** If lab install creates a printer named only `PDF24`, will the line above work?

<details>
<summary>Answer</summary>

No — Windows matches the **queue name** in quotes. Rename queue to `PDFCreator` or change VBA.

</details>

## Success criteria

- [ ] You can explain “name vs driver” in your own words
- [ ] You know what to install on lab VM for Steris AC exports
