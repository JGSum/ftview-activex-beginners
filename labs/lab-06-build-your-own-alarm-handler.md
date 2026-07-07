# Lab 6 — Build your own alarm FOR-loop handler

**Goal:** Reproduce the Decon **`Alarm_Act`** pattern in a tiny program so `[DINT_IX].[BIT_IX]` stops feeling like magic.

**Time:** ~45 min in Studio 5000 + Logix Emulator (or your lab PLC).

**Reference:** [Lesson 6 — PLC alarm FOR loop](../lessons/06-plc-alarm-for-loop-indirection.md)

---

## Part A — Tags (Controller scope)

Create these tags:

| Tag | Type | Notes |
|-----|------|-------|
| `Lab_Alarm_ACT` | DINT[0] | 32 bits — live conditions |
| `Lab_Alarm_ON` | DINT[0] | 32 bits — latched events |
| `Lab_Alarm_ACK` | DINT[0] | 32 bits — ack from HMI or pushbutton |
| `Lab_Alarm_ACT_AND_ACKED` | DINT[0] | Internal state |
| `Lab_HMI_Banner` | DINT | 0=none, 1=unacked, 2=acked |
| `Lab_ALARM_IX` | DINT | FOR loop index |
| `Lab_DINT_IX` | DINT | 0 for alarms 1–31 |
| `Lab_BIT_IX` | DINT | Bit number within DINT |
| `Lab_DEACT_DELAY` | TIMER | 1 s off-delay |
| `Lab_Ack_PB` | BOOL | Momentary ack button (or force bit) |

Optional inputs to **simulate** Alarm 22-style conflict:

| Tag | Type |
|-----|------|
| `Lab_Pressure` | REAL |
| `Lab_ATM_High` | REAL |
| `Lab_ATM_Low` | REAL |
| `Lab_Switch_Hi` | BOOL |
| `Lab_Switch_Lo` | BOOL |

---

## Part B — Program `Lab_Alarms`

### Routine `Conditions` (direct tags — no indirection)

**Rung 0 — Alarm 1:** “Tank high”

```
XIC(Lab_Tank_High) OTE(Lab_Alarm_ACT[0].1);
```

**Rung 1 — Alarm 2:** “Pressure conflict” (mini Alarm 22)

```
[GT(Lab_Pressure, Lab_ATM_High) XIO(Lab_Switch_Lo) ,
 LT(Lab_Pressure, Lab_ATM_Low)  XIO(Lab_Switch_Hi) ,
 XIO(Lab_Switch_Hi) XIO(Lab_Switch_Lo) ]
OTE(Lab_Alarm_ACT[0].2);
```

**Rung 2 — Alarm 3:** “E-stop”

```
XIO(Lab_Estop_OK) OTE(Lab_Alarm_ACT[0].3);
```

Use simple BOOL tags or forces for inputs.

### Routine `Main`

**Rung 0:**

```
JSR(Lab_Alarm_Handler, 0);
```

**Rung 1:**

```
FOR(Lab_Alarm_Handler, Lab_ALARM_IX, 1, 3, 1);
```

(Only 3 alarms in the lab — production Decon uses 1–95.)

### Routine `Lab_Alarm_Handler` (copy the Decon shape)

**Rung 0 — Map index**

```
LIMIT(1, Lab_ALARM_IX, 31) MOVE(0, Lab_DINT_IX) MOVE(Lab_ALARM_IX, Lab_BIT_IX);
```

**Rung 1 — ACTIVATE (same logic as Decon `Alarm_Act` R1)**

```
XIC(Lab_Alarm_ACT[Lab_DINT_IX].[Lab_BIT_IX])
XIO(Lab_Alarm_ON[Lab_DINT_IX].[Lab_BIT_IX])
[ OTL(Lab_Alarm_ON[Lab_DINT_IX].[Lab_BIT_IX])
  OTU(Lab_Alarm_ACK[Lab_DINT_IX].[Lab_BIT_IX])
  MOVE(1, Lab_HMI_Banner)
  RES(Lab_DEACT_DELAY) ];
```

**Rung 2 — Delay**

```
TON(Lab_DEACT_DELAY, ?, 1000);
```

**Rung 3 — DEACTIVATE**

```
XIO(Lab_Alarm_ACT[Lab_DINT_IX].[Lab_BIT_IX])
XIC(Lab_Alarm_ACT_AND_ACKED[Lab_DINT_IX].[Lab_BIT_IX])
XIC(Lab_Alarm_ON[Lab_DINT_IX].[Lab_BIT_IX])
XIC(Lab_DEACT_DELAY.DN)
OTU(Lab_Alarm_ON[Lab_DINT_IX].[Lab_BIT_IX])
OTU(Lab_Alarm_ACT_AND_ACKED[Lab_DINT_IX].[Lab_BIT_IX]);
```

**Rung 4 — ACKNOWLEDGED**

```
XIC(Lab_Alarm_ON[Lab_DINT_IX].[Lab_BIT_IX])
XIC(Lab_Alarm_ACK[Lab_DINT_IX].[Lab_BIT_IX])
XIO(Lab_Alarm_ACT_AND_ACKED[Lab_DINT_IX].[Lab_BIT_IX])
[ OTL(Lab_Alarm_ACT_AND_ACKED[Lab_DINT_IX].[Lab_BIT_IX])
  MOVE(2, Lab_HMI_Banner) ];
```

**Rung 5 — Simple ack path (stand in for HMI A&E)**

```
XIC(Lab_Ack_PB) OTL(Lab_Alarm_ACK[0].2);
```

(Expand to ack all bits in a real project; one bit is enough for the lab.)

---

## Part C — Experiments (do in order)

### Experiment 1 — See ACT vs ON split

1. Force **`Lab_Alarm_ACT[0].2 = 1`** (or trip the pressure rung).
2. Run one scan — watch **`Lab_Alarm_ON[0].2`** latch to 1.
3. Force **`Lab_Alarm_ACT[0].2 = 0`**.
4. **`Lab_Alarm_ON[0].2`** stays 1 until ack + 1 s delay.

**Checkpoint:** You should be able to say out loud: “ACT is the condition; ON is the latch.”

### Experiment 2 — Walk the FOR loop

1. Online, open **`Lab_Alarm_Handler` Rung 1**.
2. Add **`Lab_ALARM_IX`** to the tag monitor.
3. Single-step or watch during run — index counts 1, 2, 3 each scan.
4. When **`Lab_ALARM_IX = 2`**, Rung 0 sets **`Lab_DINT_IX=0`**, **`Lab_BIT_IX=2`** — same as **`[0].2`**.

### Experiment 3 — Start interlock (optional)

Add a rung:

```
NE(Lab_Alarm_ON[0], 0) MOVE(6, Lab_Start_Disabled);
```

Trip alarm 2, ack it, watch disable 6 clear only after ON drops.

---

## Part D — Compare to production Decon

| Your lab | Decon `BX45183A_lab` |
|----------|----------------------|
| `Lab_Alarm_Handler` | `Alarm_Act` |
| `Lab_ALARM_IX` 1–3 | `ALARM_IX` 1–95 |
| `Conditions` rungs | `Alarms` program (95+ rungs) |
| `Lab_Ack_PB` | HMI `AlarmEventSummary2.AckAll()` → `Alarm22.Acked` → `Alarm_ACK[0].22` |
| `Lab_Start_Disabled = 6` | `Process.Start_disabled = 6` on `Start_control` R11 |

---

## Success criteria

- [ ] You can trace **`Lab_Alarm_ON[0].2`** back to a **condition rung**, not the FOR loop.
- [ ] You can explain why ack alone does not clear **`Lab_Alarm_ON`** immediately.
- [ ] You recognize **`[DINT_IX].[BIT_IX]`** as “alarm number N this trip through the loop.”

---

## Cleanup

Delete or disable program `Lab_Alarms` before merging any test project back to production hardware.
