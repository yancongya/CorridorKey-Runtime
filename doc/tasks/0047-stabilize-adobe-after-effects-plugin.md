# Task `0047`: Stabilize Adobe After Effects Plugin

**Status:** in_progress
**Created:** 2026-11-23
**Owner:** Runtime maintainers
**Spec ref:** doc/specs/0004-add-adobe-host-plugins.md
**ADR ref:** doc/adr/0007-add-adobe-host-plugins.md
**Board ref:**
**Depends on:** doc/tasks/0018-implement-after-effects-effect.md, doc/tasks/0020-package-adobe-plugins.md

## Context

The Adobe effect implementation builds and passes its local test gates, but the
After Effects host-smoke acceptance criteria in task 0018 remain unchecked and
the Windows RTX runtime chain is not reproducible on every developer machine.
This task records the stabilization work performed on branch `ae-stabilize`, the
evidence produced, and the remaining blocker. It does not change inference,
model selection, or product support claims.

## Acceptance Criteria

- [x] The PiPL resource and `PF_Cmd_GLOBAL_SETUP` are pinned to the same version
      and out-flag values at compile time.
- [x] An uncaught exception in the effect entrypoint produces a host-visible
      error message instead of an opaque internal error.
- [x] An Adobe-enabled release build succeeds and emits Green and Blue `.aex`
      modules with a verified embedded PiPL resource.
- [x] Focused Adobe unit tests and Adobe CMake scaffold regressions pass.
- [ ] The packaged runtime loads and renders a frame in After Effects on the
      Windows RTX track.
- [ ] Saved-project parameter round-trip and bit-depth smoke checks pass in the
      host.

## Plan

- [x] Pin generated packed version to the AE SDK `PF_VERSION` macro.
- [x] Pin global out flags to the SDK capability constants.
- [x] Surface contained host errors through `PF_OutFlag_DISPLAY_ERROR_MESSAGE`.
- [x] Make the MSVC build compile sources as UTF-8 on DBCS code pages.
- [x] Make the canonical Windows display-label fallback reachable on
      PowerShell 7.3+.
- [x] Build, test, package the Adobe payload, and stage the model set locally.
- [ ] Run the canonical RTX preparation on a host that can resolve the ONNX
      Runtime source, then repeat host smoke and record results.

## Notes

Append-only log. Date each entry. Never rewrite past entries.

### 2026-11-23

- Branch `ae-stabilize` on fork `yancongya/CorridorKey-Runtime` carries four
  commits: compile-time PiPL/SDK pinning, clang-format of that guard, the
  PowerShell 7.3+ display-label fallback fix, and the MSVC UTF-8 build fix.
- Verification performed on a local release build:
  - `scripts\windows.ps1 -Task build -Preset release -EnableAdobePlugin`
    completed with 422 of 422 targets.
  - `corridorkey_adobe_green.aex` and `corridorkey_adobe_blue.aex` were emitted;
    generated PiPL contains `AE_Effect_Version { 524289 }`,
    `AE_Effect_Global_OutFlags { (1 << 25) }`, `AE_Effect_Global_OutFlags_2
    { 5120 }`, the stable match names, and the Win64 `EffectMain` entry. The
    `MIB8` PiPL signature is present in the module bytes.
  - `[unit][adobe]` passed 700 assertions across 66 test cases.
  - `test_adobe_pipl_metadata.cmake` and `test_adobe_cmake_scaffold.ps1` passed.
  - `scripts\windows.ps1 -Task package-adobe -Track dml` produced a validated
    payload with both effect identities; the staged model set was fetched with
    the in-repo `corridorkey.exe download --variant green` command and all eight
    ONNX files matched the SHA256 values in `distribution_manifest.json`.
- Local environment gaps, recorded as differences from the documented flow:
  - vcpkg was installed shallow and lacked the pinned baseline commit; full
    history was fetched.
  - The GitHub mirror used by this machine returns 404 for
    `microsoft/onnxruntime`, so the ORT source needed by `prepare-rtx` cannot be
    resolved here. GitHub mirrors and the PyPI sdist were also unavailable.
  - The CUDA 12.9 local installer exits with `0xE0E00019` on this host.
  - Inno Setup 6 could not be downloaded through the proxy, so the online
    installer executable was not produced. The staged payload was installed
    with the in-repo install helper instead.
  - A local header-only vcpkg overlay and a curated DirectML ORT binary root
    were used to build on Windows; both are machine-local and not committed.
- Evidence level reached: build passed and local unit/regression gates passed.
  After Effects load and render validation was not run because the workstation
  had a live host session open; that check remains with the user. The RTX
  product track is not validated on this machine, so no support claim changes.

## Definition of Done

All Acceptance Criteria checked, plus:

- [ ] Local tests pass (or N/A documented in Notes)
- [ ] Code review completed (human or fresh-context reviewer per WORKFLOW section 10)
- [ ] No orphan `TODO`/`FIXME` introduced
- [ ] Status updated to `done` and Notes log closes the task
