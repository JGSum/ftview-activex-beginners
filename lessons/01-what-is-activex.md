# Lesson 1 — What is ActiveX?

## In one sentence

**ActiveX** is an older Microsoft way to embed **reusable program pieces** (controls) inside other programs — like putting a mini Excel chart inside a Word document, or a **list box** inside a FactoryTalk View display.

## What you see in the HMI world

On a FactoryTalk View screen you might have:

- Buttons, text, trends (native graphics)
- **ActiveX objects** — extra Windows components with richer behavior

Example from Steris projects: **Microsoft Forms 2.0 ListBox** on the cycle report / e-sign screen. It scrolls lines of report text.

## What ActiveX is NOT

| Misconception | Reality |
|---------------|---------|
| “ActiveX opens the PDF builder” | PDF comes from **Notepad + virtual printer** (`PDFCreator` / PDF24). ListBox only **shows text**. |
| “ActiveX is only a Studio design thing” | Runtime **clients** also load the control when the display opens. |
| “Copying one file always fixes it” | You need the right **CAB/DLL version** and the **HTTP deploy path** View expects. |

## Why it still matters

Rockwell HMIs from the 2000s–2010s often used ActiveX for lists, grids, and custom widgets. Displays in your `.gfx` / XML export still reference them:

```text
progId="Forms.ListBox.1"
fileName="FM20.DLL"
```

If Windows can’t load that DLL (missing CAB, wrong version), the display shows a **hatched gray box** or Studio throws:

```text
CAB file missing on the server: http://<PC>/RSViewActiveXControlSetup/FM20DLL.CAB
```

## Key vocabulary

| Term | Meaning |
|------|---------|
| **Control** | The widget (ListBox, etc.) |
| **DLL** | The compiled library Windows loads (`FM20.DLL`) |
| **CAB** | Compressed package Rockwell serves for auto-install |
| **progId** | Program ID string Windows uses to find the control |
| **classId** | GUID identifying the control type in the display XML |

## Check yourself

1. Name one ActiveX on your Steris e-sign display.
2. Name the component that creates the PDF file.
3. Why does the error mention a **URL** instead of only `C:\...`?

<details>
<summary>Answers</summary>

1. `ListBox1` (Forms.ListBox.1 / FM20.DLL)
2. Virtual PDF printer via `NOTEPAD /PT ... "PDFCreator"` (PDF24 driver on production)
3. View clients download/register ActiveX from the HMI server’s **web share** (`RSViewActiveXControlSetup`), not only from a local folder.

</details>

**Next:** [Lesson 2 — CAB files](02-cab-files.md)
