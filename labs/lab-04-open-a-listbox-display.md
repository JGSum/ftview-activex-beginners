# Lab 4 — Open a ListBox display in Studio

**Goal:** Verify ActiveX fix — display opens without FM20 error.

**Where:** Lab VM with View Studio and copied CAB.

## Manual path

1. Open **FactoryTalk View Studio** → Steris application (Parts Washer or 4401 server)
2. Open display **`steris_sterility_electronic_signature`** or **`81 - PRINT VIEW`**
3. If error dialog appears — read first **cause** line only; don’t click through blindly
4. If no error — confirm ListBox is a **real list**, not gray hatch pattern

## What success looks like

- Studio loads display
- ListBox visible (may be empty — no runtime data yet)
- Diagnostics List has no new ActiveX load errors

## If still failing

| Error | Next check |
|-------|------------|
| CAB missing | Lab 2 HTTP path |
| Newer version required | Re-copy CAB from production |
| Different control | Note control name in error — may need another CAB |

## Optional: compare display XML

Copies in [`reference/hmi/`](../reference/hmi/) — e.g. `steris_decon_electronic_signature.xml`, `81 - PRINT VIEW.xml`.

Search for:

```text
Forms.ListBox.1
fileName="FM20.DLL"
fileVersion=
```

Version on disk should be **≥** display expectation.

## Success criteria

- [ ] Display opens in Studio without ActiveX error
- [ ] You can point to ListBox on screen
