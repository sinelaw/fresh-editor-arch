# Maintainer: Noam Lewis
#
# SPDX-License-Identifier: 0BSD
#
# Source package - builds from source.
#
# If this is ever promoted to [extra]: drop aarch64 (Arch Linux ARM is a
# separate project), set the receipt channel to "pacman", and consider adding
# check() { cargo test --frozen --workspace --lib } — omitted here because in
# the AUR that dev-profile rebuild is paid by every user on every install.

pkgname=fresh-editor
pkgver=0.4.7
pkgrel=1
pkgdesc="A lightweight, fast terminal-based text editor with LSP support and TypeScript plugins"
url="https://sinelaw.github.io/fresh/"
license=("GPL-2.0-only")
arch=('x86_64' 'aarch64')
depends=("gcc-libs" "glibc" "hicolor-icon-theme" "xz")
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
    # The tree pins a toolchain in rust-toolchain.toml; override it so rustup
    # users build with their stable instead of downloading the pinned version.
    export RUSTUP_TOOLCHAIN=stable
    # Deliberately not --target-filtered: the filtered fetch omits crates the
    # build still resolves (platform-gated ones), which makes build()'s
    # --frozen fail. Costs a larger download, buys a truly offline build.
    cargo fetch --locked
}

build() {
    cd "fresh-$pkgver"
    export RUSTUP_TOOLCHAIN=stable
    export CARGO_TARGET_DIR=target
    export CC=clang
    # Bake the install channel into the binary (compile-time provenance).
    export FRESH_BUILD_CHANNEL=aur
    # --frozen keeps the build offline and pinned; it only holds because
    # prepare() fetches the whole lockfile rather than one target's slice.
    # Default features on purpose, and never --all-features: that would enable
    # fresh-update/insecure-endpoints, which must never ship in a real build.
    cargo build --frozen --release
}

package() {
    cd "fresh-$pkgver"

    install -Dm755 target/release/fresh "$pkgdir/usr/bin/fresh"

    # Documentation
    install -Dm644 README.md "$pkgdir/usr/share/doc/$pkgname/README.md"

    # License
    install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

    # Provenance receipt: found relative to the binary, so /usr/bin/fresh
    # resolves it here. Keeps self-update from touching a pacman-owned file.
    install -dm755 "$pkgdir/usr/share/$pkgname"
    cat > "$pkgdir/usr/share/$pkgname/install-receipt.toml" <<EOF
schema = 1
channel = "aur"
version = "$pkgver"
package_name = "fresh-editor"
managed = true
self_update = false

[hints]
aur_pkg = "fresh-editor"
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
