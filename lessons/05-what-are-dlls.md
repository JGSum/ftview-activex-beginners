# Lesson 5 — What are DLLs? (and why ActiveX cares)

## In one sentence

A **DLL** (Dynamic Link Library) is a **shared code file** Windows loads when a program needs a feature it doesn’t bundle itself — like a plugin the OS keeps in a known place.

## Everyday analogy

| Piece | Role |
|-------|------|
| **FactoryTalk View Studio** | The main program (opens displays) |
| **FM20.DLL** | A **library** that knows how to draw a Microsoft Forms **ListBox** |
| **Windows** | Finds and loads `FM20.DLL` when View asks for `Forms.ListBox.1` |

View doesn’t contain ListBox code inside its own `.exe`. It says: *“Windows, give me the ListBox control”* → Windows loads **`FM20.DLL`**.

## Why DLLs matter for ActiveX

ActiveX controls on HMI displays are often **separate DLLs** (or OCXs):

```text
Display XML  →  progId="Forms.ListBox.1"  fileName="FM20.DLL"
```

So when ActiveX fails, you’re usually debugging **which DLL**, **which version**, and **which copy** Windows actually loaded — not the display graphics themselves.

## Three places the “same” DLL can live

This is the mental model for **what we fixed on the lab VM**:

| Location | What it is | Our lab story |
|----------|------------|---------------|
| **Inside the CAB** | Shipped copy (`16.0.10357.20081`) | Copied from production — **correct version** |
| **IIS / inetpub URL** | Download path for View to fetch CAB | Fixed 404 → browser could download |
| **`C:\Windows\SysWOW64\FM20.DLL`** | **The copy 32-bit programs actually use** | Was **`11.0.8000`** (ancient) → caused “newer version required” |

View Studio is **32-bit**. It does **not** use the 64-bit `System32` copy for this — it uses **`SysWOW64`**.

CAB download + IIS can be perfect and Studio still fails if **SysWOW64** has an old `FM20.DLL`.

## Version numbers

Right-click DLL → **Properties** → **Details** → **File version**.

Display project XML may record:

```text
fileVersion="16.0.10357.20081"
```

Windows must load a DLL **at least that new** (or the exact build the project expects). Mismatch → *“Newer file version of ActiveX control required.”*

## CAB vs DLL — how they relate

```text
FM20DLL.CAB  ──extract──►  FM20.DLL + FM20.inf
       │
       └──IIS serves CAB──► View tries auto-setup
                                    │
                                    └──often fails to replace old SysWOW64 copy
```

**CAB** = delivery package. **DLL** = what actually runs. You need both paths understood.

## FM20ENU.DLL (bonus)

`FM20ENU.DLL` next to `FM20.DLL` in `SysWOW64` is the **English strings** helper for Forms. Usually leave it alone when swapping `FM20.DLL`.

## What we did today (lab VM checklist)

1. Copied **CAB** to Rockwell folder + **inetpub** (IIS URL).
2. Verified CAB **version** matches display (`16.0.10357.20081`).
3. Found **SysWOW64\FM20.DLL** was still **11.0.8000**.
4. Closed Studio → renamed old DLL → copied new **FM20.DLL** into **SysWOW64**.
5. **E-sign display opened clean.**

## Check yourself

1. Why check SysWOW64 if the CAB version was already correct?
2. What’s the difference between a CAB and a DLL?

<details>
<summary>Answers</summary>

1. Because 32-bit View loads the DLL from SysWOW64 at runtime; CAB setup doesn’t always replace an older system file.
2. CAB = compressed delivery box; DLL = the actual code Windows executes for the control.

</details>

**Lab:** [Lab 5 — SysWOW64 FM20 swap](../labs/lab-05-syswow64-fm20.md)
