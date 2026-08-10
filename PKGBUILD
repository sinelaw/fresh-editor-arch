# Maintainer:
# Contributor: Noam Lewis
#
# SPDX-License-Identifier: 0BSD
#
# [extra] variant of the AUR fresh-editor package. Differences from the AUR
# version: x86_64 only, receipt channel "pacman", and a check().

pkgname=fresh-editor
pkgver=0.4.7
# Promotion bump, so AUR users upgrade into the repo version cleanly.
pkgrel=2
pkgdesc="A lightweight, fast terminal-based text editor with LSP support and TypeScript plugins"
url="https://sinelaw.github.io/fresh/"
license=("GPL-2.0-only")
arch=('x86_64')
depends=("gcc-libs" "glibc")
makedepends=("cargo" "clang")
conflicts=("fresh-editor-bin")
# Mixed Rust/C project: makepkg's lto option puts -flto in CFLAGS, so clang
# emits bitcode that rustc's bundled lld refuses whenever their LLVM majors
# differ. Only the C objects lose LTO; profile.release still does fat LTO.
options=('!lto')
source=("fresh-editor-${pkgver}-source.tar.gz::https://github.com/sinelaw/fresh/releases/download/v${pkgver}/fresh-editor-${pkgver}-source.tar.gz")
sha256sums=("e313fdf0eb01aa8b5c024218241a153999ec9c8ba538ad1ba41f407c884b22c0")

prepare() {
    cd "fresh-$pkgver"
    # The tree pins a toolchain in rust-toolchain.toml; override it so the
    # build uses the packaged stable rather than the pinned version.
    export RUSTUP_TOOLCHAIN=stable
    # Deliberately not --target-filtered: the filtered fetch omits crates the
    # build still resolves (platform-gated ones), which makes --frozen fail
    # below. Costs a larger download, buys a truly offline build.
    cargo fetch --locked
}

build() {
    cd "fresh-$pkgver"
    export RUSTUP_TOOLCHAIN=stable
    export CARGO_TARGET_DIR=target
    export CC=clang
    # Bake the install channel into the binary (compile-time provenance).
    export FRESH_BUILD_CHANNEL=pacman
    # --frozen keeps the build offline and pinned; it only holds because
    # prepare() fetches the whole lockfile rather than one target's slice.
    # Default features on purpose, and never --all-features: that would enable
    # fresh-update/insecure-endpoints, which must never ship in a real build.
    cargo build --frozen --release
}

check() {
    cd "fresh-$pkgver"
    export RUSTUP_TOOLCHAIN=stable
    export CARGO_TARGET_DIR=target
    export CC=clang
    # Unit tests only: the integration suites want a TTY, network and a real
    # $HOME, none of which a clean chroot provides.
    cargo test --frozen --workspace --lib
}

package() {
    cd "fresh-$pkgver"

    install -Dm755 target/release/fresh "$pkgdir/usr/bin/fresh"

    # Documentation
    install -Dm644 README.md "$pkgdir/usr/share/doc/$pkgname/README.md"

    # License
    install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

    # Provenance receipt: found relative to the binary, so /usr/bin/fresh
    # resolves it here. Tells the editor pacman owns this install, so the
    # self-update path defers instead of writing to /usr.
    install -dm755 "$pkgdir/usr/share/$pkgname"
    cat > "$pkgdir/usr/share/$pkgname/install-receipt.toml" <<EOF
schema = 1
channel = "pacman"
version = "$pkgver"
package_name = "fresh-editor"
managed = true
self_update = false
EOF

    # Plugins, themes and keymaps are compiled into the binary (embed-plugins
    # and include_str!), so no copy on disk is read at runtime.

    # Desktop file
    install -Dm644 crates/fresh-editor/resources/fresh.desktop "$pkgdir/usr/share/applications/fresh.desktop"

    # Hicolor icons
    for icon in docs/icons/linux/hicolor/*/apps/fresh.png; do
        size=$(basename "$(dirname "$(dirname "$icon")")")
        install -Dm644 "$icon" "$pkgdir/usr/share/icons/hicolor/${size}/apps/fresh.png"
    done
}
