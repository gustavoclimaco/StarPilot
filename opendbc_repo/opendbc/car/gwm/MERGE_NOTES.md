# GWM Haval H6 GT port - upstream merge notes

The port is deliberately additive so that merges from `upstream/StarPilot` stay cheap.

## New files (zero conflict surface)

    opendbc/car/gwm/{__init__,values,fingerprints,interface,carstate,carcontroller,gwmcan,radar_interface}.py
    opendbc/dbc/gwm_haval_h6_mk3.dbc
    opendbc/safety/modes/gwm.h

## Shared files touched - re-apply these if a merge reverts them

| File | Change |
|---|---|
| `opendbc/car/car.capnp` | `gwm @37;` at the end of `enum SafetyModel` |
| `opendbc/safety/declarations.h` | `#define SAFETY_GWM 37U` |
| `opendbc/safety/safety.h` | `#include "opendbc/safety/modes/gwm.h"` + `{SAFETY_GWM, &gwm_hooks},` |
| `opendbc/car/values.py` | `from opendbc.car.gwm.values import CAR as GWM` + `GWM` in the `Platform` union |
| `opendbc/car/torque_data/params.toml` | `"GWM_HAVAL_H6" = [1.8, 1.5, 0.15]` |
| `CREDITS.md` | provenance section |

Note `opendbc` is a symlink to `opendbc_repo/opendbc`, so the real paths git tracks are
under `opendbc_repo/`.

**Safety id 37 is ours, not upstream's.** If upstream adds its own `@37` / `37U`, renumber
the GWM entry in `car.capnp` and `declarations.h` together - they must agree.

## Known deltas from the FrogPilot original

- `make_can_msg()` returns `(addr, dat, bus)` here, so checksum helpers index `[1]`, not `[2]`.
- `CarInterface._update` no longer exists; the EPS "silently refusing" counter moved into
  `carstate.py` and reads `self.CC.latActive` (populated by `selfdrive/car/card.py`).
- Per-brand events live in `selfdrive/car/car_specific.py`, dispatched on `CP.brand`. No GWM
  branch is registered there yet - the port currently relies on the generic common events.
- `fwd_hook` was dropped: all five spoofed messages are declared with `.check_relay = true` and
  the generic forwarder blocks the stock copies. Each stock copy arrives on the opposite bus from
  where openpilot transmits, so relay-malfunction cannot false-trigger.
- RX checks are fail-closed in this codebase: `ignore_checksum` / `ignore_counter` /
  `ignore_quality_flag` must be set explicitly or the check resolves to invalid.
