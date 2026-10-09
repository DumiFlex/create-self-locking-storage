# Self-locking vault storage (Create 6)

A storage system that **never sends a package to the vaults unless it's sure the package has somewhere to
land**. It fills 18 max-size vaults from one buffer, slows itself down as the vaults get full, stops when they
are, and starts again by itself as soon as you take something out. No packages stuck on belts, nothing lost,
and the buffer's items stay reachable through the stock network the whole time.

---

## Quick start (the schematic)

**Download:** the schematic is [`schematics/create_self_locking_storage.nbt`](../schematics/create_self_locking_storage.nbt) in this repository, and on
[createmod.com](https://createmod.com/schematics/self-locking-package-storage-18-vaults).

**After placing the schematic, do these before using it.** Signs in the build point to each spot.
1. **Set every threshold switch.** Placing a schematic resets them, and with reset switches the system sends
   packages to vaults that can't take them. That's 3 switches on each of the 18 vaults (54 in total). The signs
   **on the back** mark which column of switches is **A**, **B** and **C**. Set each switch to count **stacks**,
   leave the filter empty, turn **Invert on**, and enter the top and bottom values from the table below.
2. **Drop one item on the belt clock** at the top (the small 8-block belt loop, marked with a sign). Without an
   item circling on it there are no clock pulses and nothing is ever sent. One item only: two would double
   every clock.
3. **Set up the frogports:**
   - The **6 input frogports** at the front (marked with signs): set them to **send and receive**, and give all 6
     the **same name**, for example `Storage` or `Main Storage`. That's the address the rest of your network
     sends items to.
   - **Every other frogport** (the ones with a stock link): **no name**, set to **send only**.
4. Connect rotation, at **256 RPM** if you can.

The redstone and the redstone link frequencies come through the schematic intact; only the threshold switches
need setting.

**Threshold switch settings** (every vault, both sides; each vault holds 1,620 stacks). **Every switch must have
Invert turned on**, count in **stacks**, and have an empty filter. Without Invert a switch signals when the vault
is *full* instead of when it has room, and the system sends packages to full vaults.

| Switch | Speed it controls | Default: safe at any RPM | Only if you always run at 256 RPM |
| --- | --- | --- | --- |
| A | Fast | top **1,432**, bottom **1,431** | top 1,558, bottom 1,557 |
| C | Medium | top **1,567**, bottom **1,566** | top 1,585, bottom 1,584* |
| B | Slow | top **1,612**, bottom **1,611** | same |

\* Worked out from test results, not tested on its own yet.

**Use the default column** unless you're sure the speed never changes. The 256 column is only safe at exactly
256 RPM; the same build at 128 RPM needs more free space per vault than that.

**Best speed: 256 RPM.** Above about 80 RPM it fills just as fast (the packagers can't send faster than one
package every 2 seconds anyway), and at 256 RPM the fewest packages are on the belts at once. **Avoid 72–80
RPM**: the packagers are right on the edge of their own speed limit there and behave unpredictably. Slower
speeds are safe with the default settings, they just fill more slowly.

**What you'll see** (every light has a sign next to it)
- Filling goes fast, slows down near the end, then stops. Take items out and it starts again by itself.
- **4 lights at the top**, one per common packager (2 per side). Each flashes with the same pulse that makes its
  packager send, so you can watch the system work: both lights of a side flashing quickly = **fast**; only one
  of them flashing, every 4th clock lap = **medium**; only one, every 16th lap = **slow**; dark = that side is
  full or there's nothing to send.
- **Storage full light** at the front: on when no vault on either side has room for another package. New items
  then wait in the buffer, where you can still request them.
- Up to 8 stacks per vault can stay empty. That's on purpose: a package can be 9 stacks, so a vault with less
  room than that doesn't get one.

**Don't change** belt lengths, chutes, the number of vaults or packagers, the 8-block belt clock, or the
redstone, or the numbers above stop being correct. If you do, see "Changing the build" at the end.

---

## How it works

### The problem

Storage with many vaults usually sends each package to a specific vault. But a package is sent *before* it
arrives, and by the time it gets there the vault may already be full: other packages were on their way to it
too. Those packages then have nowhere to go and sit on the belt forever, and their items can't be requested
while they're stuck.

### The idea

Packages go down one line past all 9 vaults on a side. At each vault a funnel tries to put the package into
the vault's packager, and **a packager only accepts a package if the whole thing fits**. If it doesn't, the
package just carries on to the next vault. Nothing decides in advance where a package goes: it lands in the
first vault with room.

That only leaves one question: **how do we make sure a package we send will fit somewhere?** We can't check
when it's already on the belt. So instead, every vault **keeps some space free**, and the buffer stops sending
before that space is gone. The space kept free has to fit every package that might already be on its way.

### Why keep space free (the reserve)

Imagine the network is busy: the buffer is full and the packagers are sending packages one after another, as
fast as they can. The first package needs a few seconds to travel down the line. By the time it lands in a
vault, more packages are already following it.

At 256 RPM we counted how many: when the first package lands and the vault's threshold switch notices,
**5 more** are already on the way. Each package can be up to 9 stacks, so they can need up to 45 stacks of
room. Add one more pulse of 2 packages as a safety margin and you get **7 packages = 63 stacks**.

So switch A says: "while this vault has **at least 63 stacks free**, keep sending at full speed". When the
vault gets fuller than that, the switch turns off and the buffer stops. The packages that were already on the
way still fit into those 63 stacks. Nothing gets stuck.

At other speeds more packages can be on the way at once (up to 15 around 72–78 RPM), which is why the default
setting keeps **189 stacks** free instead: that covers every speed.

**When the network is quiet** and packages only come now and then, fewer are on the way at once. But the system
can't tell how busy it will be in the next few seconds, so it always plans for the busy case.

### Why three speeds

Keeping 63 (or 189) stacks free in every vault would waste a lot of space. So when a vault gets close to its
reserve, the system doesn't just stop: it **switches to a slower speed**. Sending less often means fewer
packages can be on the way at once, so the vault needs less free space, and it can fill further.

| Speed | Sends | Packages on the way at once | Space kept free (default) |
| --- | --- | --- | --- |
| **Fast** | 2 packages per clock pulse (both packagers) | up to 15 + safety | 189 stacks |
| **Medium** | 1 package every 4 pulses (one packager) | up to 4 + safety | 54 stacks |
| **Slow** | 1 package every 16 pulses (one packager) | exactly 1 | 9 stacks |

Each vault has three threshold switches, one per speed. As a vault fills up:
1. While it has more than its fast reserve free, **fast** runs.
2. Below that, switch A turns off and the system drops to **medium**.
3. Below the medium reserve, it drops to **slow**: one package at a time, so the vault only needs room for one.
4. Below 9 stacks free (less than one package), it **stops**.

This happens automatically, based only on the threshold switches. Take items out and the switches turn back on,
so it speeds up again by itself.

**In practice the slow speeds only matter at the very end.** Packages always go into the first vault with room.
While any vault still has lots of space, the system stays on fast, and the packages fill vault 1 all the way to
the top, then vault 2, and so on, because a passing package still drops into any vault that has room for it.
Only when the *last* vault with space gets close to full does the system slow down.

### Waiting when it slows down

When fast speed stops, the fast packages are still on the belt. If medium started straight away, it would count
on space those packages are about to take. So after switching down, each slower speed **waits one full trip**
(long enough for every package on the belt to land) before it sends anything. In the redstone these are the
"armed" latches.

### The clock

Everything is timed by a small belt loop with one item circling on it: every lap gives one pulse. Medium and
slow use the same pulses divided by 4 and by 16.

Why a belt and not a normal redstone clock? Because the belt loop runs on the same rotation as the storage
line. If someone lowers the RPM, the packages travel slower, and the clock slows down by exactly the same
amount, so packages never bunch up on the belt. With a fixed-time clock, a slow belt would fill up with more and
more packages, and no reserve would be big enough.

At high RPM the clock is actually faster than the packagers can send (a packager only sends once every 2
seconds), so the extra pulses are simply ignored. That's why 256 RPM fills at full speed with the fewest
packages on the belt.

### Two sides, one buffer

The storage has two mirrored sides of 9 vaults each. They only share the **buffer**, the **return belt** and
the **storage full lamp**; everything else is separate (packagers, belts, threshold switches, link frequencies,
redstone). Each side locks and unlocks on its own, so when one side is full, everything goes to the other side.

### Safety net

After each side's last vault, the belt loops back into the buffer. If a package ever does get past every vault
(it shouldn't, but timing isn't perfect), it simply goes back into the buffer and gets sent again later,
usually through the other side. Nothing is ever lost.

---

## Settings for every speed

The numbers come from testing the built storage. For each speed: run fast mode with a full buffer, and the
moment the first package lands in the last vault and its threshold switch reacts, count every package still on
its way.

| RPM | Packages still on the way | Fast reserve (+1 pulse safety) | Switch A top / bottom | Medium reserve | Switch C top / bottom |
| --: | --: | --: | --: | --: | --: |
| 256 | 5 | 7 packages = 63 | 1,558 / 1,557 | 36 | 1,585 / 1,584 |
| 192 | 5 | 7 = 63 | 1,558 / 1,557 | 36 | 1,585 / 1,584 |
| 128 | 7 | 9 = 81 | 1,540 / 1,539 | 36 | 1,585 / 1,584 |
| 96 | 7–9 | 11 = 99 | 1,522 / 1,521 | 36 | 1,585 / 1,584 |
| 72–80 | 7 or 15 (varies) | 19 = 171 | 1,450 / 1,449 | 27 | 1,594 / 1,593 |
| 48–64 | 13 | 15 = 135 | 1,486 / 1,485 | 27 | 1,594 / 1,593 |
| 16–32 | 11 | 13 = 117 | 1,504 / 1,503 | 27 | 1,594 / 1,593 |
| **Any speed (default)** | | **21 = 189** | **1,432 / 1,431** | **54** | **1,567 / 1,566** |

Slow (switch B) is the same at every speed: top 1,612, bottom 1,611 (one package, 9 stacks).

**Why does it go up and then down again?**
- **At high RPM** the packagers' 2-second limit decides how often packages leave, and the trip is short, so
  few are on the way.
- **Going slower,** the packagers still send as fast as they can, but each trip takes longer, so more packages
  are on the way at once.
- **Around 72–80 RPM** one clock lap takes about as long as a packager's 2-second limit. Depending on how they
  happen to line up, a run sends on every pulse (15 on the way) or on every other pulse (7). That's why the
  same speed gave different results.
- **Below that,** the clock is slower than the packagers, so sending slows down together with the belts and the
  count drops again.

**Why the medium numbers are what they are:** medium sends one package every 4 laps from one packager. From the
fast counts we know how many laps a trip takes at each speed (between about 5.5 and 12.5), so at most 2–3 medium
packages are on the way, plus 1 for safety. The default **54** was what the build was tested with and has extra
margin; the lower values are worked out, not tested.

**Why the switch values are "top = capacity − (reserve − 1), bottom = capacity − reserve":** the switch should
stay on while the vault has *at least* its reserve free. With 1,620 stacks and a 9-stack reserve, a vault
holding 1,611 still has room for exactly one package, so the switch has to stay on at 1,611 and turn off at
1,612.

**Warnings**
- A per-speed setting is only safe at that speed. The numbers don't go steadily up or down with RPM: a build set
  for 256 RPM is not safe at 128 or 64 RPM. If players can change the speed, use the default row.
- The numbers are for this exact build. Change the line and you need to test again.
- Medium and slow must only ever drive **one** packager. Their reserves count one package per pulse.

---

## Building it

### Layout (one side)

```
            INCOMING PACKAGES (frogports)
                       │
                 ┌─────▼─────┐
  return ──────► │  BUFFER   │ ◄── stock link (low priority)
  packager       └─────┬─────┘
                       │ 2 common packagers ◄── gates
                       ▼
  belts/chutes ─► [S1] ─► [S2] ─► ... ─► [S9] ─► shared return belt ─► buffer
                   │       │              │
                  V1      V2             V9
                   └───────┴──── ... ─────┘
                     switch B on every vault (both sides)
                                │ SLOW
                                ▼
                    [inverse extender] ─► STORAGE FULL lamp

  [S] = 2 funnels → 2 unpowered packagers → vault
```

- **Buffer:** a 3×3×7 vault (1,260 stacks), the storage's input. Incoming frogports unpack into it. Its stock
  link has **lower priority** than the vaults, so its items are always requestable but the vaults are used
  first. Its size makes the belts leaving it the right length for the numbers above.
- **Common packagers:** 2 per side (4 on the buffer), **no sign** (these packages need no address). Each drops
  its packages onto a 2-block branch belt; the two branches merge with tunnels onto the main line.
- **Main line:** 32 belts after the tunnels and 6 chutes, in an S shape past the 9 vaults.
- **Vault stations:** 2 funnels per vault, each into an **unpowered** packager stuck to the vault. Two, so that
  two packages arriving close together both get a chance at the same vault.
- **Vaults:** max size 3×3×9, 1,620 stacks each.
- **Return belt:** after the last vault of each side, into the buffer through a funnel and an unpowered packager.

### Clock

- **Belt clock:** 4 belts of 2 blocks in a square, each dropping onto the next, so one lap is 8 blocks. One item
  on it. Same rotation and RPM as the storage line. A smart observer watches one belt, followed by a pulse
  repeater on its shortest setting: one short pulse per lap.
- **Divider:** a chain of powered toggle latches, each halving the pulses.
  - after latch 2: **÷4**, through a pulse repeater = **medium** pulse
  - after latch 4: **÷16**, through a pulse repeater = **slow** pulse

### Threshold switches and links

Three per vault, each with a redstone link next to it. Every switch: count in **stacks**, empty filter,
**Invert on** (so it outputs while the vault still has room).
- **A → FAST**, **C → MEDIUM**, **B → SLOW**, one frequency each, the same on every vault of a side. The other
  side uses three different frequencies.
- Links on the same frequency combine: FAST is on if **any** vault on that side has its fast reserve free.
- **Keep every switch isolated.** Threshold switches also power the blocks around them. Put glass or a gap
  between a switch and any link that isn't its own, including on the next vault, or one switch can turn on
  another switch's link.

### Gates

Each gate decides when a pulse is allowed to reach the packagers. They're built from **pulse extenders**, all
set to **2 ticks**: the input extenders all output into one shared dust line, which feeds an output extender O.
O is always **inverse**. For each input: if the signal **must be on**, use an **inverse** extender; if it
**must be off**, use a **normal** one.

| Gate | Inputs | Drives |
| --- | --- | --- |
| Fast | fast pulse (inverse), FAST (inverse) | both packagers |
| Medium | ÷4 pulse (inverse), MEDIUM (inverse), FAST (normal), medium ARMED (inverse) | packager 1 |
| Slow | ÷16 pulse (inverse), SLOW (inverse), FAST (normal), MEDIUM (normal), slow ARMED (inverse) | packager 1 |

```
 FAST GATE
 fast loop ─[PR]─[I]─●
          (FAST)─[I]─●
                     ●─[I]─[R]─┬─[R]─► Packager 1
                       (O)     └─[R]─► Packager 2

 MEDIUM GATE
 ÷4 pulse ──[PR]─[I]─●
           (MED)─[I]─●
          (FAST)─[N]─●
     medium ARMED────[I]─●
                     ●─[I]─[R]──► Packager 1

 SLOW GATE
 ÷16 pulse ─[PR]─[I]─●
          (SLOW)─[I]─●
          (FAST)─[N]─●
           (MED)─[N]─●
       slow ARMED────[I]─●
                     ●─[I]─[R]──► Packager 1

 [I] inverse extender   [N] normal extender   [PR] pulse repeater   [R] repeater   ● shared dust line
```

- A **repeater** in front of each packager keeps the signals one-way, so medium and slow can never reach
  packager 2.
- Keep the gates apart (a gap or glass), so their dust lines don't connect.
- Put link receivers directly behind the extenders, so no dust is needed on the inputs.

**Test with levers** in place of the receivers and a lamp on each gate's output: with FAST on, only the fast
gate flashes; with FAST off and MEDIUM on, only medium (after its wait); with only SLOW on, only slow (after its
wait); with all off, nothing.

### Armed latches (the wait after slowing down)

```
 Medium:
  STEP 1  (÷16)──[N 4t]──► [latch M1] ──► step 2               side ◄ (FAST)
  STEP 2  (÷16)──[I]─●
                     ●──[I]──[N 4t]──► step 3
          M1 ────[I]─●
  STEP 3  from step 2 ──► [latch M2] ──► medium gate's ARMED   side ◄ (FAST)

 Slow:
  (÷16)──[N 4t]──► [latch] ──► slow gate's ARMED               side ◄ (FAST) + (MEDIUM)
```

- A powered latch turns **on** from its back and **off** from its side.
- **Medium waits for two slow pulses** after fast stops. It sends every 4 laps, so after only one slow pulse it
  could start too early.
- **Slow waits for one.** It only sends on slow pulses itself, so skipping the first one is already a full trip.
- The **4-tick delays** ([N 4t] = normal extender set to 4 ticks) must stay longer than the 2-tick gate pulses,
  so the pulse that arms a latch can't also send a package.
- The gates still check FAST and MEDIUM themselves: a latch held off from the side still flickers on for an
  instant when pulsed from the back, and the gate checks stop that flicker from sending anything.

### Storage full lamp

Both sides' **SLOW** receivers power the input of one **inverse** pulse extender, which lights the lamp. SLOW is
on while any vault on that side has room for a package, so the lamp lights only when **neither side** can take
another one, and stays lit until something is taken out. An observer on the return belt can't do this job: the
system locks itself before packages ever reach the return belt.

---

## Testing

| Test | Result |
| --- | --- |
| One pulse sends the right number of packages (fast 2, medium 1, slow 1) | passed |
| Slow pulse is longer than a trip (package reaches the last vault before the next slow pulse), at 256 RPM | ÷8 failed, **÷16 passed** |
| Fill from empty at 256 RPM: vaults fill in order, it slows down at the end, every vault ends full, lamp on | **passed** |
| Take items out of a full vault: sending restarts by itself and refills that gap | passed |
| Request items from the buffer while storage is full | passed |
| Packages on the way, measured at 13 speeds from 16 to 256 RPM | done, see Settings for every speed |
| Medium stress test: fill everything, then take 100–150 stacks out of **every** vault, check nothing reaches the return belt | **still to do** (needed before using medium below 54) |
| Fill test at 72–78 RPM (the busiest speed) | still to do |

### Problems found while building

| What went wrong | Why | Fix |
| --- | --- | --- |
| A slow pulse sent 2 packages | both packagers were wired to every gate | fast → both packagers; medium and slow → packager 1 only, with repeaters in front |
| Extra packages near the end | threshold switches powered other switches' links | isolate every switch (glass or gaps) |
| Slow speed overflowed by 1 package at 256 RPM | the slow pulse (÷8) came before the package arrived; chutes don't speed up with RPM | ÷16 |
| Overflow right after fast stopped | slower speed started while fast packages were still landing | armed latches |
| Last vault stopped one package short | switch values one stack off | top = capacity − (reserve − 1) |
| Results at 72–80 RPM jump between 7 and 15 | the clock lap is about as long as the packager's 2-second limit | keep the default reserve, avoid running at 72–80 RPM |

---

## Changing the build

The reserves depend on the exact build. If you change it (more vaults, longer belts, more chutes, another
packager count), recalculate:

**Quick estimate before you can test** (comes out too high, never too low; this build estimates 21 where the
real worst case was 15):

    packages on the way = (belts after the tunnels ÷ 8, rounded up) × packagers per pulse
                        + branch belt blocks + chute blocks + packagers + 1
    fast reserve = that × 9 stacks

- Count belts up to the last funnel of the last vault.
- Branch blocks and chutes count as 1 each because packages can bunch up there; main-line belts count 1 per 8
  blocks because packages are spaced one clock lap apart.
- Lengths in the same 8-block chunk cost the same: 25–32 belts all count as 4.

**Better: measure.** Block the belt just after the last vault, run fast mode with a full buffer, and count the
packages when the first one reaches the end. Do it at several speeds, especially the one where the belt clock
flashes once per package (that's the busiest). Highest count + one pulse of safety, × 9 = fast reserve.

**Medium:** trip length in laps ÷ 4, rounded up, + 1 safety, × 9. **Slow:** always 9, as long as the slow pulse
is longer than a trip (repeat the ÷16 race test at max RPM; add a toggle latch for ÷32 if it fails).

**Bigger storage:** don't lengthen a line, add another side or line on the buffer instead. Each new line needs
its own packagers, belt, vaults, redstone and link frequencies, and keeps the same numbers if it's built the
same. The limit is how many faces the buffer has free.

---

## Why not other designs

- **Frogports with addresses per vault** decide where a package goes before it arrives. With 4 packages on their
  way to a vault that only has room for 1, three get stuck.
- **A plain loop back to the buffer** never gets stuck, but packages that fit nowhere keep going round, and their
  items can't be requested while they're travelling.
- **Waiting for each package to land before sending the next** (a smart observer on all 18 vault funnels) is
  exact, but expensive in survival and slow, because only one package can be on the way.
