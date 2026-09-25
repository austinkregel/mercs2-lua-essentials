# The Ess.UI movies — src/shipment/movies/

`Ess.UI` does not draw with engine primitives. Every widget is a clip inside a Scaleform movie, and
the game has to be able to load those movies by name. The movies are committed in this repo as
`src/shipment/movies/*.gfx`, and they reach the game **only through the Quartermaster Shipment**:
`manifest.yaml` declares one `add_movie` per file, and `qm` builds them into the game.

**The OnLoad zip (`Ess-<version>.zip`) carries no movies, so `Ess.UI` does not draw on an OnLoad
install.** `data/vz-patch.wad`, the patch WAD that used to carry them in that zip, is gone.

The only movie the kit actually uses is **`ess_ui`** — one runtime movie whose AS2 payload draws
every widget from theme parameters. `Ess.UI.FILES` (`src/42_ui_engine.lua`) also lists the
pre-rewrite per-widget movies (`ui_list`, `ui_panel`, `ui_bar`, `ui_toast`, `ui_confirm`, `ui_input`,
`chat`, `contracts`), kept because `Ess.UI.FILES.panel` and friends are published surface a
third-party script may reference. `forge` is loaded by `samples/demos/MissionForge.lua`. `cbar` and
`cpanel` were in the WAD and ship too.

## Where the files came from

All 12 were extracted **byte-exact** from `data/vz-patch.wad` as of commit `3430c0b`, the last commit
that had it. Each extracted movie was checked: it is an uncompressed GFX v8 file whose first tag is
ExporterInfo, the name in that tag equals the file name, and `pandemic_hash_m2(name)` equals the WAD
asset hash it came from. Re-wrapping each movie reproduced its WAD block byte for byte. Each file's size
and sha256 are recorded in the commit that added it.

## The gotcha that shipped a broken release

**Assets are registered under their bare stem, but loaded with the extension.** `Ess.UI.FILES` says
`"ess_ui.gfx"` and `SetSwfFile` takes `"ess_ui.gfx"`, yet the asset is registered as
`pandemic_hash_m2("ess_ui")` — the extension is stripped before hashing. That is why every
`add_movie` `name:` in `manifest.yaml` is the bare stem (`ess_ui`, not `ess_ui.gfx`). A movie
registered under `ess_ui.gfx` is a name the engine will never look up, and the failure is **silent**:
the widget host constructs fine, so nothing errors, nothing logs, and the UI simply never appears.

v0.5.1 shipped without `ess_ui` at all — the WAD had been committed once, before the UI rewrite, and
`package.py` only checked that the file existed. It did not reproduce in development because the dev
install had the movie injected by hand.

## Updating a movie

`ess_ui.gfx` is authored in **gfxforge-web** (`examples/mercs2/ess_ui.js` compiles to
`examples/mercs2/ess_ui.gfx`), not in this repo. A rebuild there does not touch anything here: to
ship a new version, **replace the committed `src/shipment/movies/ess_ui.gfx`** with the new export
and commit it. There is no WAD to re-inject. To add a movie, commit the `.gfx` under
`src/shipment/movies/` and add a matching `add_movie` entry (bare-stem `name:`) to `manifest.yaml`.

This repo has no movie gate of its own. `qm lint` confirms that each declared movie file exists, and
`qm build` that it parses as a movie; nothing checks that the movie is the current export or that
`Ess.UI.FILES` and `manifest.yaml` agree. Check a changed movie in the game.
