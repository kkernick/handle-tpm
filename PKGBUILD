pkgname=handle-tpm-git
pkgdesc="Seal secrets in the TPM"
pkgver=r13.9534805
pkgrel=1

source=("git+https://github.com/kkernick/handle-tpm.git")
sha256sums=("SKIP")
depends=(systemd tpm2-tools openssl coreutils bash)
arch=("any")
provides=("handle-tpm")

pkgver() {
	cd $srcdir/handle-tpm
	printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short=7 HEAD)"
}

package() {
	cd $srcdir/handle-tpm
	for binary in *.tpm; do
		install -Dm755 "$binary" "$pkgdir/usr/bin/$binary"
	done
}
