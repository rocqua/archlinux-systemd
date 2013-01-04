# Maintainer: Dave Reisner <dreisner@archlinux.org>

pkgname=systemd-git
pkgver=20130104
pkgrel=1
pkgdesc="system and service manager"
arch=('i686' 'x86_64')
url="http://www.freedesktop.org/wiki/Software/systemd"
license=('GPL2' 'LGPL2.1' 'MIT')
depends=('acl' 'bash' 'dbus-core' 'glib2' 'kbd' 'kmod' 'libcap' 'libgcrypt' 'pam' 'util-linux' 'xz')
makedepends=('docbook-xsl' 'git' 'gobject-introspection' 'gperf'
             'gtk-doc' 'intltool' 'libxslt' 'python2')
optdepends=('cryptsetup: required for encrypted block devices'
            'python2: systemd library bindings'
            'python2-cairo: systemd-analyze'
            'python2-gobject: systemd-analyze'
            'quota-tools: kernel-level quota management')
provides=('systemd' 'libsystemd' 'systemd-sysvcompat' 'systemd-tools' 'udev=999')
conflicts=('systemd' 'libsystemd' 'systemd-sysvcompat' 'systemd-tools' 'sysvinit' 'initscripts' 'udev')
groups=('systemd')
options=('!libtool')
backup=(etc/dbus-1/system.d/org.freedesktop.systemd1.conf
        etc/dbus-1/system.d/org.freedesktop.hostname1.conf
        etc/dbus-1/system.d/org.freedesktop.login1.conf
        etc/dbus-1/system.d/org.freedesktop.locale1.conf
        etc/dbus-1/system.d/org.freedesktop.timedate1.conf
        etc/systemd/system.conf
        etc/systemd/user.conf
        etc/systemd/logind.conf
        etc/systemd/journald.conf
        etc/udev/udev.conf)
install='systemd.install'
source=('use-split-usr-path.patch'
        'initcpio-hook-udev'
        'initcpio-install-udev'
        'initcpio-install-timestamp'
	'fstab-overmount.patch')
md5sums=('fd5b5f04ab0a847373d357555129d4c0'
         'e99e9189aa2f6084ac28b8ddf605aeb8'
         'fb37e34ea006c79be1c54cbb0f803414'
         'df69615503ad293c9ddf9d8b7755282d'
	 '1a4551879bd71c3685f1a49b7915f86a')

_gitroot="git://anongit.freedesktop.org/systemd/systemd.git"
_gitname="systemd"

build() {
  msg "Connecting to GIT server...."

  if [[ -d $_gitname ]] ; then
    cd $_gitname && git pull origin
    msg "The local files are updated."
  else
    git clone "$_gitroot" "$_gitname"
  fi

  msg "GIT checkout done or server timeout"
  msg "Starting make..."

  rm -rf "$srcdir/$_gitname-build"
  git clone "$srcdir/$_gitname" "$srcdir/$_gitname-build"
  cd "$srcdir/$_gitname-build"

  patch -Np1 <"$srcdir/use-split-usr-path.patch"
  patch -Np1 <"$srcdir/fstab-overmount.patch"

  ./autogen.sh
  ./configure \
      PYTHON=python2 \
      PYTHON_CONFIG=/usr/bin/python2-config \
      --libexecdir=/usr/lib \
      --localstatedir=/var \
      --sysconfdir=/etc \
      --enable-introspection \
      --enable-gtk-doc \
      --disable-audit \
      --disable-ima \
      --with-distro=arch \
      --with-sysvinit-path= \
      --with-sysvrcd-path=

  make
}

package() {
  make -C "$_gitname-build" DESTDIR="$pkgdir" install

  printf "d /run/console 0755 root root\n" >"$pkgdir/usr/lib/tmpfiles.d/console.conf"

  # compat symlinks
  install -dm755 "$pkgdir"/sbin
  for tool in runlevel reboot shutdown poweroff halt telinit; do
    ln -s '/usr/bin/systemctl' "$pkgdir/sbin/$tool"
  done
  ln -s '../usr/lib/systemd/systemd' "$pkgdir/sbin/init"

  # fix .so links in manpage stubs
  find "$pkgdir/usr/share/man" -type f -name '*.[[:digit:]]' \
      -exec sed -ri '1s|^\.so (.*)\.([0-9]+)|.so man\2/\1.\2|' {} +

  # move bash-completion and symlink for loginctl
  install -Dm644 "$pkgdir/etc/bash_completion.d/systemd-bash-completion.sh" \
      "$pkgdir/usr/share/bash-completion/completions/systemctl"
  for ctl in {login,journal,timedate,locale,hostname}ctl; do
    ln -s systemctl "$pkgdir/usr/share/bash-completion/completions/$ctl"
  done
  rm -rf "$pkgdir/etc/bash_completion.d"

  # don't write units to /etc by default
  rm "$pkgdir/etc/systemd/system/getty.target.wants/getty@tty1.service"
  rmdir "$pkgdir/etc/systemd/system/getty.target.wants"

  # get rid of RPM macros
  rm -r "$pkgdir/etc/rpm"

  # can't use py3k yet with systemd-analyze -- the 'plot' verb will not work.
  # https://pokersource.info/show_bug.cgi?id=50989
  sed -i '1s/python$/python2/' "$pkgdir/usr/bin/systemd-analyze"

  # Replace dialout/tape/cdrom group in rules with uucp/storage/optical group
  sed -i -e 's#GROUP="dialout"#GROUP="uucp"#g' \
         -e 's#GROUP="tape"#GROUP="storage"#g' \
         -e 's#GROUP="cdrom"#GROUP="optical"#g' "$pkgdir"/usr/lib/udev/rules.d/*.rules

  # add mkinitcpio hooks
  install -Dm644 "$srcdir/initcpio-install-timestamp" "$pkgdir/usr/lib/initcpio/install/timestamp"
  install -Dm644 "$srcdir/initcpio-install-udev" "$pkgdir/usr/lib/initcpio/install/udev"
  install -Dm644 "$srcdir/initcpio-hook-udev" "$pkgdir/usr/lib/initcpio/hooks/udev"
}
