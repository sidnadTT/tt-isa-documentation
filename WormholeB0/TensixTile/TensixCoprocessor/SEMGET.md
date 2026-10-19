# `SEMGET` (Decrement the `Value` of some semaphores)

**Summary:** Various semaphores are selected using a bitmask, and the selected semaphores have their `Value` decremented (unless their `Value` is already zero, in which case it remains at zero).

**Backend execution unit:** [Sync Unit](SyncUnit.md)

## Syntax

```c
TT_SEMGET(/* u8 */ SemaphoreMask)
```

## Encoding

![](../../../Diagrams/Out/Bits32_SEMGET.svg)

## Functional model

```c
atomic {
  for (unsigned i = 0; i < 8; ++i) {
    if (SemaphoreMask.Bit[i] && Semaphores[i].Value > 0) {
      Semaphores[i].Value -= 1;
    }
  }
}
```

## Instruction scheduling

If the programmer intent is for the semaphore decrement to happen after some earlier Tensix instruction has _finished_ execution, it may be necessary to insert a [`STALLWAIT`](STALLWAIT.md) instruction (with block bit B1) somewhere between that earlier instruction and the `SEMGET` instruction.

If the issuing thread's most recent [`SEMWAIT`](SEMWAIT.md) selects a semaphore that this instruction changes, its block mask does not include B1, and no instruction blocked by that mask has been issued by the thread since, then the `SEMWAIT` might still be latched when this instruction executes, and whether it is depends on timing. As this instruction can change the condition the `SEMWAIT` is waiting on, this is UndefinedBehavior: include B1 in the `SEMWAIT`'s block mask.
