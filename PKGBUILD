pkgname=handle-tpm-git
pkgdesc="Seal secrets in the TPM"
pkgver=2
pkgrel=1

source=("git+https://github.com/kkernick/handle-tpm.git")
sha256sums=("SKIP")
depends=(systemd tpm2-tools openssl coreutils bash)
arch=("any")
provides=("handle-tpm")

package() {
	cd $srcdir/handle-tpm
	for binary in *.tpm; do
		install -Dm755 "$binary" "$pkgdir/usr/bin/$binary"
	done
}
