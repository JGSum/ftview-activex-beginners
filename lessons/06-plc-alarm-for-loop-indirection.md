# Lesson 6 — PLC alarm FOR loop and `[DINT_IX].[BIT_IX]` indirection

**From real troubleshooting:** Steris Decon lab, `Alarm_ON[0].22` blocking start (`Start_disabled = 6`).

## The confusion in one line

You watch **`Alarm_ON[0].22`** in the tag monitor, but the ladder shows **`Alarm_ON[DINT_IX].[BIT_IX]`**. Those are the **same bit** — the FOR loop is just visiting every alarm number through a pointer.

---

## Three tags for one alarm

| Tag | Role | Who sets it |
|-----|------|-------------|
| **`Alarm_ACT[n].b`** | Condition is **true right now** | Alarm condition rungs in `Alarms` (direct tag, e.g. `Alarm_ACT[0].22`) |
| **`Alarm_ON[n].b`** | **Latched** “alarm event active” | `Alarm_Act` subroutine, Rung 1 — **OTL** only |
| **`Alarm_ACK[n].b`** | Operator acknowledged | HMI A&E ack → `Alarm22.Acked` → PLC rung copies to `Alarm_ACK[0].22` |

**Start interlocks read `Alarm_ON`, not `Alarm_ACT`.** Acknowledging on the HMI does not instantly clear `Alarm_ON`.

---

## Scan order (Decon `BX45183A_lab`)

Every PLC scan, in order:

### 1. Condition rungs — `Alarms` program

Example: **Alarm 22 — Pressure transmitter conflict** (`Alarms` Rung 47):

```
Pressure vs ATM limits conflict with DIG_IN[0].0 / DIG_IN[0].1
AND Process.Step > 1
→ (after Alarm_Delay[22]) OTE(Alarm_ACT[0].22)
```

Mode B lab: `Input_test[0]` → `DIG_IN[0]` when `Option.Test_DI` is on.

**This rung uses real bit numbers.** No indirection here.

### 2. FOR loop — `Alarms_main` Rung 1

```
FOR(Alarm_Act, ALARM_IX, 1, 95, 1);
```

Calls subroutine **`Alarm_Act`** **95 times per scan** with `ALARM_IX = 1, 2, 3 … 95`.

### 3. Index translation — `Alarm_Act` Rung 0

When `ALARM_IX = 22`:

| ALARM_IX | → DINT_IX | → BIT_IX |
|----------|-----------|----------|
| 1–31     | 0         | = alarm # |
| 32–63    | 1         | alarm # − 32 |
| 64–95    | 2         | alarm # − 64 |

So for alarm 22: **`DINT_IX = 0`**, **`BIT_IX = 22`**.

Now `[DINT_IX].[BIT_IX]` means `[0].22`.

### 4. Activate — `Alarm_Act` Rung 1 (ACTIVATE ALARM)

```
XIC(Alarm_ACT[DINT_IX].[BIT_IX])
XIO(Alarm_ON[DINT_IX].[BIT_IX])
→ OTL(Alarm_ON[DINT_IX].[BIT_IX])
→ OTU(Alarm_ACK[DINT_IX].[BIT_IX])
→ MOVE(1, HMI.Alarm_banner)
```

**This is the only place in the program that sets `Alarm_ON`.**

Fires once per alarm: when ACT turns on and ON was still off.

### 5. Acknowledge — `Alarm_Act` Rung 4

After HMI ack (`Alarm_ACK` bit set):

```
XIC(Alarm_ON) XIC(Alarm_ACK) XIO(Alarm_ACT_AND_ACKED)
→ OTL(Alarm_ACT_AND_ACKED)
→ MOVE(2, HMI.Alarm_banner)
```

### 6. Deactivate — `Alarm_Act` Rung 2 + 3

- **R2:** `TON(DEACT_DELAY, 1 sec)`
- **R3:** When ACT is **off**, acked, ON still on, delay done:
  - **`OTU(Alarm_ON)`** — bit finally clears
  - Print “ALARM OFF” line

---

## Trace backward from `Alarm_ON[0].22`

```
Alarm_ON[0].22 = 1
    ↑ OTL on Alarm_Act R1 when ALARM_IX=22
Alarm_ACT[0].22 was 1 at some point
    ↑ OTE on Alarms R47 (pressure vs ATM switches)
DIG_IN[0].0 / .1 disagree with PT549 OR pressure out of band
    ↑ Mode B: Input_test[0].0 / .1
```

**The FOR loop did not create the condition.** It only copied ACT → ON.

---

## Why HMI “ACTIVE ALARM” and `Start_disabled` disagree

| Layer | What it uses |
|-------|----------------|
| Cycle Status banner | FTView **`AE_*Count`** (alarm server) |
| PLC start block | **`Alarm_ON[0/1/2]`** whole DINTs + **`HMI.Alarm_banner = 1`** + **`Cycle_aborted`** |

You can ack in A&E (banner yellow/solid) while **`Alarm_ON[0].22`** is still latched for ~1 second (or longer if ACT is still true).

---

## Lab-only note (Curia Decon emulator)

`docs/plc-changes-lab-only.md` documents **`AFI()`** on `Alarms` Rung 47 to bypass Alarm 22 in the lab L5X. If the **running emulator** was not downloaded from that file, R47 still runs and ACT can keep re-firing.

---

## Mental model

Think of **`Alarm_ACT`** as the **sensor** (live) and **`Alarm_ON`** as the **event latch** (for interlocks and printing). The FOR loop is a **factory** that processes all 95 alarms the same way instead of duplicating five rungs × 95 alarms.

---

## Next

Do **[Lab 6 — Build your own alarm handler](../labs/lab-06-build-your-own-alarm-handler.md)** in Logix Emulator or a spare routine — same pattern, three alarms only.
