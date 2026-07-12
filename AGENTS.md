# Agent instructions — ftview-activex-beginners

## What this repo is

A **teaching repo** — lessons and hands-on labs for FactoryTalk View SE, ActiveX, and related **Allen-Bradley PLC** patterns.

Content is grounded in the **Curia ABQ Steris simulation lab** (Decon autoclave + Parts Washer on FactoryTalk Logix Echo). That is the **live system students learn on**, not a generic tutorial VM.

This repo holds **explanations and lab steps**. The running lab, session logs, and production promotion lists live in **[Curia_ABQ](https://github.com/JGSum/Curia_ABQ)**.

## What this repo is not

- **Not** the lab infrastructure or backup kit. `Curia_ABQ/Lab_Back_up/` is for **disaster recovery** (restore the simulation elsewhere). Do **not** treat it as a prerequisite for doing labs or wire new lessons to it.
- **Not** a place to commit production ACDs, HMI `.apb` files, or large binaries unless the user explicitly asks.
- **Not** a substitute for Rockwell manuals — cite verified paths and tags from Curia ABQ docs or L5X exports.

## When helping the user

### Teaching mode (default)

- **One step at a time.** Wait for their result before the next instruction.
- **Lead with the concept**, then the single next action on the Steris lab.
- **PLC-first** when the topic is reports, alarms, or sequencing — HMI and PDF24 are the thin top layer.
- **Check yourself** blocks in lessons are for the student; when tutoring live, ask one question instead of dumping answers.

### Writing new lessons or labs

- Anchor examples to **verified Steris lab artifacts**: `BX45183A_lab`, `GMP_Lab_lab`, Echo `.70` / `.100`, `4401_HMI_Server` / `ABQ4401FTV`, tag names from `reference/plc/*.L5X` or `Curia_ABQ/ACDs/*.L5X`.
- Pair every new **lesson** with a **lab** when hands-on practice makes sense.
- Number sequentially (`08`, `lab-08-…`). Update `README.md` tables.
- Cross-link **Curia_ABQ** docs for operator flows and session-verified facts — not `Lab_Back_up`.

### Accuracy

- Do not invent FT View menu paths, tag names, or routine names. Grep `reference/plc/*.L5X`, `reference/hmi/*.xml`, or read `Curia_ABQ/docs/` first.
- If the Steris lab differs from production, say **lab vs production** explicitly.

### Scope

- Stay in this repo for lesson/lab **content**.
- Implementation changes (PLC edits, HMI exports, exe builds) belong in **Curia_ABQ** unless the user asks otherwise.

## Key references (Steris system truth)

| Topic | Where |
|-------|--------|
| Operator cycle + report flow | `Curia_ABQ/docs/lab-cycle-simulation-and-report-process.md` |
| Decon step-by-step | `Curia_ABQ/docs/lab-decon-cycle-stepthrough.md` |
| Echo / network / PLC conversion | `Curia_ABQ/docs/lab-echo-ftv-session-2026-07.md` |
| L5X for ladder study | `reference/plc/` (this repo) · `Curia_ABQ/ACDs/` |
| HMI display XML | `reference/hmi/` (this repo) · `Curia_ABQ/DisplaysXMLs/` |
| HMI / PDF / ActiveX session work | `Curia_ABQ/docs/lab-setup-plan.md`, `docs/ftview-vba-exports.md` |
| Restore lab on new VMs (rare) | `Curia_ABQ/Lab_Back_up/LAB_SETUP_INSTRUCTIONS.txt` — **not** for routine lab work |

## Lab environment (what students use)

| Item | Typical lab value |
|------|-------------------|
| HMI VM | FactoryTalk View SE client + Studio |
| Decon PLC (Echo) | `BX45183A_lab` @ `192.168.100.70` |
| PW PLC (Echo) | `GMP_Lab_lab` @ `192.168.100.100` |
| HMI project | `4401_HMI_Server` — Decon + PW clients |
| PDF queue | **`PDFCreator`** (PDF24 driver) |

Students need the **running Steris lab** (or equivalent they already have). They do not need to unpack `Lab_Back_up` to study Lesson 6 or run Lab 7.
