# Building ideal.net (Zen fork) on Windows

Verified 2026-07-13 on Windows 11 Pro (10.0.26200), 8-core x86_64, building
**Zen 1.21.6b / Firefox 152.0.5** from `upstream/stable`, zero custom patches.

This documents the *working* path on this machine, including where it deviates
from Zen's own docs. Zen's CI cross-compiles Windows builds from Ubuntu+wine;
the native Windows path below is NOT what upstream exercises daily — expect
papercuts on Zen updates, all known ones are listed here.

## Required versions (pinned by Zen in-repo)

| Tool | Version | Pinned in | Notes |
|---|---|---|---|
| Firefox | 152.0.5 | `surfer.json` | source tarball, ~800 MB download |
| Rust | 1.90 (**msvc**, not gnu) | `.rust-toolchain` | `rustup default 1.90-x86_64-pc-windows-msvc` |
| Node | 22.x | `.nvmrc` | 24.x worked fine in practice |
| Python | 3.11 | `.python-version` | 3.13.14 worked fine in practice; needs `pip install -r requirements.txt` |
| clang/LLVM | fetched by `mach bootstrap` | mozilla toolchain | do NOT install your own; lands in `~/.mozbuild/clang` |
| surfer | ^1.14.6 | `package.json` | Zen's fork-management CLI, from npm |

## One-time machine setup

1. **Visual Studio Build Tools 2022** with:
   - `Microsoft.VisualStudio.Workload.VCTools` (C++ build tools, MSVC 14.4x, Win11 SDK)
   - `Microsoft.VisualStudio.Component.VC.ATL` ← **not installed by default; Firefox hard-requires it**
   - (for future aarch64: `VC.Tools.ARM64` + `VC.ATL.ARM64`)
2. **MozillaBuild** (Latest) → `C:\mozilla-build`
   (`https://ftp.mozilla.org/pub/mozilla/libraries/win32/MozillaBuildSetup-Latest.exe /S`)
3. **Rust msvc toolchain**: `rustup toolchain install 1.90-x86_64-pc-windows-msvc --profile minimal && rustup default 1.90-x86_64-pc-windows-msvc`
4. **Windows Defender exclusion** for the workspace directory
   (`Add-MpPreference -ExclusionPath "C:\Workspace"`, elevated). Without it,
   every filesystem-heavy step is 5–20× slower over the ~520k-file tree.
5. **Disk**: tarball 0.8 GB + source 4.3 GB + engine git baseline ~1.5 GB +
   objdir ~40 GB → keep **≥ 60 GB** free.

## Build steps (in order)

Run everything from **PowerShell or cmd — never Git Bash** (deviation #1 below).

```powershell
git clone <our-fork> ideal.net; cd ideal.net
npm ci
python -m pip install -r requirements.txt

# -- deviation #2: do NOT use `npm run download` (deadlocks, see below). Instead:
npx surfer download          # ok to let it fetch the tarball, then Ctrl-C / kill node
                             # when it hangs after "Unpacking Firefox..."
mkdir engine
tar -xf .surfer\engine\firefox-152.0.5.source.tar.xz -C engine --strip-components=1

# -- engine git baseline (surfer export diffs against this; ~10 min with the
#    config below, HOURS without it)
cd engine
git config core.autocrlf false; git config core.safecrlf false
git config core.fscache true;   git config core.preloadindex true
git config core.untrackedCache true; git config gc.auto 0; git config commit.gpgsign false
cd ..
npx surfer ff-init engine      # git init + orphan branch "152.0.5" + baseline commit

npm run import                 # builds tools/ffprefs (Rust) + applies all 242 Zen patches

# -- deviation #3: surfer can't spawn `mach` on Windows; call it directly:
cd engine
python mach bootstrap --application-choice=browser --no-system-changes

# -- the build itself (surfer build == writes engine/mozconfig, then ./mach build)
cd ..
npx surfer build               # let it write engine/mozconfig; it then dies at mach —
cd engine                      # finish manually:
$env:ACCEPTED_MAR_CHANNEL_IDS="unofficial"; $env:MAR_CHANNEL_ID="unofficial"
python mach build

# run it
python mach run --noprofile    # (== `npm start` from repo root)
```

## Windows deviations from Zen's docs (root causes)

1. **Never build from Git Bash.** Git's GNU `/usr/bin/link.exe` shadows the
   MSVC linker; Rust builds (e.g. `tools/ffprefs` during `npm run import`)
   fail with `link.exe ... extra operand`. PowerShell resolves `link` to MSVC.
2. **`npm run download` deadlocks silently** after "Unpacking Firefox...".
   surfer's win32 path runs `7z x` without `-y`, so 7z blocks forever on an
   interactive overwrite prompt (node sits at 0% CPU). Extract the tarball
   manually with Windows' built-in bsdtar (see steps above) — this mirrors
   surfer's own non-Windows code path (`--strip-components=1`).
3. **`surfer bootstrap` / the mach step of `surfer build` fail instantly**
   with cmd's "'mach' is not recognized": surfer spawns the extensionless
   `mach` script, which Windows can't execute. Run the underlying
   `python mach <cmd>` yourself in `engine/` (surfer's own `engine/mach.cmd`
   shim does the same).
4. **Git performance on the 517k-file engine tree.** A global
   `core.autocrlf=input` made the baseline `git add` take **8.5 hours**
   (line-ending inspection of every file + Defender scanning each). With the
   per-repo config above + Defender exclusion it's minutes. Symptom if you
   regress: `git` process pegged with slowly-growing CPU and no index file.
5. `mach` suggests moving the tree to a ReFS Dev Drive for 5–10% faster
   builds; optional, suppress with `MACH_HIDE_DEV_DRIVE_SUGGESTION=1`.
6. **A killed build can leave corrupt generated headers that survive
   incremental rebuilds.** Symptom: `error: no member named '...' in
   namespace 'mozilla::StaticPrefs'` even though the pref exists in
   `modules/libpref/init/StaticPrefList.yaml`. Cause: the header-install
   step was interrupted, leaving e.g.
   `obj-*/dist/include/mozilla/StaticPrefs_<group>.h` full of whitespace
   with a fresh timestamp, so make never regenerates it. Fix: compare with
   the valid copy in `obj-*/modules/libpref/StaticPrefs_<group>.h` and copy
   it over (or delete the blank file and rebuild). Long builds are best run
   detached from any agent/terminal session that might be restarted.

## Approximate timings on this machine (8-core, NVMe, Defender-excluded)

| Step | Time |
|---|---|
| tarball download (800 MB) | ~10 min (network-bound) |
| bsdtar extract | ~33 min |
| `ff-init` baseline commit | ~10 min (tuned git) / 8.5 h (untuned — don't) |
| `npm run import` | ~15 min (incl. first ffprefs cargo build) |
| `mach bootstrap` | ~20 min (multi-GB toolchain downloads) |
| `mach build` (dev, no LTO/PGO) | ~8 h of CPU work on 8 cores; budget a full day. (Measured across interrupted incremental runs; final resume alone was 12.6 h wall at 95% CPU with the machine in shared use.) |

## Fork/git structure

- `origin` = our repo, `upstream` = `https://github.com/zen-browser/desktop`.
- `main` is based on **`upstream/stable`** (decision: track stable, not dev).
  Update flow: `git fetch upstream && git rebase upstream/stable`.
- Zen's layout keeps forks rebase-friendly and we inherit it:
  - `src/zen/**` — Zen's own code, in a directory vanilla Firefox doesn't
    have (never conflicts). **Our code will live in `src/ideal/**` the same way.**
  - `src/<firefox-path>/*.patch` — surgical diffs against vanilla Firefox
    files, applied by `surfer import`, regenerated by `surfer export <file>`.
  - `engine/` (gitignored) — the patched Firefox tree; its internal git repo
    holds the vanilla baseline commit that `surfer export` diffs against.
- Only `src/`, `prefs/`, `configs/`, `surfer.json` etc. are tracked; the
  browser engine itself is always reproducible from tarball + patches.

## Build flavors

Local default is a **dev build** (surfer `buildMode: dev` — `--disable-debug`,
no `ZEN_RELEASE`, no LTO/PGO, unofficial branding). Release flags
(`ZEN_RELEASE=1`, LTO, PGO, official branding) are CI concerns; see
`configs/common/mozconfig` and `.github/workflows/windows-release-build.yml`
(note: upstream's Windows CI cross-compiles from Ubuntu with wine).
