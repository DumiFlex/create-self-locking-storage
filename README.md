# Self-locking vault storage for Create 6

A storage system that **never sends a package to the vaults unless it's sure the package has somewhere to
land**. It fills 18 max-size vaults from one buffer, slows itself down as the vaults get full, stops when they
are, and starts again by itself as soon as you take something out. No packages stuck on belts, nothing lost,
and the buffer's items stay reachable through the stock network the whole time.

**Download:** [`schematics/create_self_locking_storage.nbt`](schematics/create_self_locking_storage.nbt), or on
[createmod.com](https://createmod.com/u/2ab947cd61936e5b757f7d2d24f74991).

## Quick start

After placing the schematic (signs in the build point to each spot):
1. **Set all 54 threshold switches** (3 per vault). Placing a schematic resets them. The signs on the back mark
   the **A**, **B** and **C** columns. Every switch: count in **stacks**, empty filter, **Invert on**, then:

   | Switch | Speed | Top | Bottom | Only if you always run at 256 RPM |
   | --- | --- | --- | --- | --- |
   | A | Fast | 1,432 | 1,431 | top 1,558, bottom 1,557 |
   | C | Medium | 1,567 | 1,566 | top 1,585, bottom 1,584* |
   | B | Slow | 1,612 | 1,611 | same |

   These are safe at any RPM. If you always run at exactly 256 RPM, see the full docs for tighter settings.
2. **Drop one item on the belt clock** at the top (the small 8-block belt loop). One item only.
3. **Frogports:** the 6 input frogports at the front: **send and receive**, all with the **same name** (for
   example `Storage`). Every other frogport (the ones with a stock link): **no name**, **send only**.
4. Connect rotation, at **256 RPM** if you can. Avoid 72–80 RPM.

The redstone and link frequencies survive the schematic; only the threshold switches need setting.

**Lights:** the 4 lights at the top flash with each common packager (fast: both lights of a side flashing
quickly; medium: one, every 4th lap; slow: one, every 16th lap). The storage full light at the front comes on
when no vault can take another package; new items then wait in the buffer, where you can still request them.

## Full documentation

[docs/storage_system.md](docs/storage_system.md) explains how it works, why there are three speeds, why the
thresholds are what they are, the settings for every speed, how it's built, how it was tested, and how to
recalculate if you change the build.
