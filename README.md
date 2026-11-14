# platform-ios

iOS toolchain bundles for [TSClang](https://github.com/tsclang/tsclang):

* **libuv** — **built from source** (#287): libuv v1.53.0 cross-compiled
  with cmake into one host-independent mobile asset,
  `libuv-1.53.0-mobile.tar.gz` (release `toolchains-v1`), darwin runner
  only (the iOS slices need Xcode).

The bytes are identical to the `toolchains-v2` asset of
[platform-android](https://github.com/tsclang/platform-android): the
mobile bundle is one artifact from one build job, duplicated into this
repo so that every platform package points at its own repo (follow-up to
#287). The android slices ride along in the same archive — a
cross-compiled static library does not depend on the machine that runs
the compiler, and splitting the artifact would fork the build recipe.

The compiler itself is NOT bundled: Apple SDKs are not redistributable,
so iOS compilation goes through `xcrun --sdk <sdk> clang` and requires
macOS with Xcode Command Line Tools (#283). `libuv` is needed by the
LINKER — `-luv` for `--async libuv` — not by the compiler.

Installed by `tsclang platform install @tsclang/ios` into
`~/.tsclang/platforms/…`, sha256-pinned in the platform manifest.

libuv layout (`packages/registry/scripts/build-bundles.mjs` in the
compiler repo, entry `libuv`):

```
include/            ← uv.h + uv/*.h (from the source tree, arch-independent)
lib/arm64-apple-ios16.0/libuv.a              ─┐ CMAKE_SYSTEM_NAME=iOS,
lib/arm64-apple-ios16.0-simulator/libuv.a    │ deployment target 16.0
lib/x86_64-apple-ios16.0-simulator/libuv.a   ─┘
lib/aarch64-linux-android28/libuv.a   ─┐ android slices (same archive as
lib/x86_64-linux-android28/libuv.a     ┘ platform-android — see above)
```

The `lib/<triple>/` names are exactly the `--target` triples TSClang
compiles with, so the linker line resolves as
`-L <root>/lib/<triple> -luv` (android adds `-ldl`).

## Targets

| bundle | os-arch | status |
|--------|---------|--------|
| libuv | any host | built from source on a darwin runner (`bundles.yml`) |

Releases are immutable per tag (`toolchains-v1`, `toolchains-v2`, …):
assets are never overwritten — consumers have the sha256 pinned. A new
libuv revision = new asset names + manifest update + `.tspkg` republish
in the compiler repo.

License: libuv — MIT, built from the official `libuv-v1.53.0.tar.gz`
source tarball (noted in the bundle's `TSC_BUNDLE_NOTICE.txt`).
