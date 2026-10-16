# SwiftVLC fork maintenance

## Upstream rebase, 2026-10-16

- Original fork main: `7859814d2a3d3a67a74412573296916de76e1fa4`.
- Original common base: `192393a45cdcd7ebb6befac26586360b8b1676b3`.
- New upstream main: `0ec31e4ecd31d08189b5aa64f86dc00f1434c978`
  (`harflabs/SwiftVLC`, v1.1.0-beta.14).
- Recovery branch: `codex/pre-rebase-swiftvlc-20261016`.

| Fork functionality | Decision |
| --- | --- |
| iOS 16 support | Retain Perception, mutex backport, native deployment policy, and sample-buffer API fallbacks. Adapt new upstream flags and diagnostics to iOS 16. |
| AirPlay surface access | Retain `PiPVideoView.onSurfaceReady`, the reparented-view layout guard, and `Player.refreshVideoOutputAfterDisplayMove()`. Add surface access to the new preferred initializer. |
| Native PiP restoration | Use upstream's generation-aware delegate bridge and bounded restore coordinator. Remove the fork delegate proxy. |
| Active PiP external binding | Retain the controller binding while PiP survives dismantling the inline view. Upstream still clears it unconditionally. |
| Xcode 27 sample-buffer isolation | Retain the protocol-based enqueue call needed by the legacy display-layer fallback. |
| Native deployment verification | Use upstream's stronger per-object Mach-O metadata validation, configured for iOS 16. Remove the duplicate fork check. |
| v1.0.1 release pointers | Drop both obsolete commits. Keep upstream's current native engine instead of reverting to an ABI-incompatible old binary. |
| Fork release tooling | Retain fork publication URLs, accept both official and fork artifact identities, and resolve authenticated downloads from the artifact owner's repository. |

Textual conflicts occurred in the observation-backed player/equalizer/PiP
classes, native PiP backend, build/release scripts, and obsolete release
references. Resolutions retain upstream's playback, renderer, lifecycle, and
release invariants while adapting only the fork functionality above.

The old native compatibility patch cannot precede upstream's new renderer
patches. `0055-ios16-sample-buffer-renderer-compat.patch` now runs after all 54
upstream patches. It selects the display layer on iOS 16 and the separate
renderer on iOS 17+, including matching recovery-notification identities.

## Validation and native artifact boundary

All 2,457 Swift tests in 209 suites pass on macOS. The iOS Simulator library
build at deployment target 16.0 passes with Xcode 27.1. All 55 native patches apply to VLC
`c833c4be000b426d73ff4324bec574065f00e3df` and pass the complete upstream
native source contract runner, including extension version 12, lifecycle race
models, recovery, seeking, audio leases, and adversarial mutation checks.
The native compatibility API surface also compiles for arm64 iOS 16 with
unguarded-availability errors enabled. Release-reference round trips and
setup-dev transformation tests pass.

These are source/build checks, not physical-device playback validation.
The complete XCFramework was not rebuilt or published during the rebase.
`Package.swift` deliberately retains the current official engine URL and
checksum, whose iOS deployment floor is 18. It does not establish iOS 16 binary
support. Before distributing for iOS 16, build a new fork XCFramework with
`./scripts/build-libvlc.sh --clean-build --all`, qualify it on devices, and
publish it through the fork release process. The existing local `Vendor/`
artifact was preserved; do not use its old engine for validating current
native features.
