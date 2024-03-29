# Maintainer: Fredy García <frealgagu at gmail dot com>

pkgname=jdk-bcl
pkgver=8u202
pkgrel=1
pkgdesc="Oracle Java Development Kit (BCL)"
arch=("x86_64")
url="https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html"
license=("custom:Oracle")
depends=("ca-certificates-java" "hicolor-icon-theme" "java-environment-common" "java-runtime-common" "nss" "xdg-utils")
optdepends=(
  "alsa-lib: for basic sound support"
  "eclipse-java: to use \"Oracle Java Mission Control\" plugins in Eclipse"
  "gtk2: for Gtk+ look and feel (desktop)"
)
provides=(
  "java-environment=${pkgver%u*}"
  "java-environment-jdk=${pkgver%u*}"
  "java-openjfx=${pkgver%u*}"
  "java-runtime=${pkgver%u*}"
  "java-runtime-headless=${pkgver%u*}"
  "java-runtime-headless-jre=${pkgver%u*}"
  "java-runtime-jre=${pkgver%u*}"
  "java-web-start=${pkgver%u*}"
  "java-web-start-jre=${pkgver%u*}"
)
backup=(
  "etc/java-${pkgname}/amd64/jvm.cfg"
  "etc/java-${pkgname}/images/cursors/cursors.properties"
  "etc/java-${pkgname}/management/jmxremote.access"
  "etc/java-${pkgname}/management/management.properties"
  "etc/java-${pkgname}/security/java.policy"
  "etc/java-${pkgname}/security/java.security"
  "etc/java-${pkgname}/security/javaws.policy"
  "etc/java-${pkgname}/content-types.properties"
  "etc/java-${pkgname}/flavormap.properties"
  "etc/java-${pkgname}/fontconfig.properties.src"
  "etc/java-${pkgname}/logging.properties"
  "etc/java-${pkgname}/net.properties"
  "etc/java-${pkgname}/psfont.properties.ja"
  "etc/java-${pkgname}/psfontj2d.properties"
  "etc/java-${pkgname}/sound.properties"
)
options=(!strip) # JDK debug-symbols
install="${pkgname}.install"
source=(
  "manual://${pkgname%-bcl}-${pkgver}-linux-x64.tar.gz"
  "https://download.oracle.com/otn-pub/java/jce/${pkgver%u*}/jce_policy-${pkgver%u*}.zip"
  "jconsole-${pkgname}.desktop"
  "jmc-${pkgname}.desktop"
  "jvisualvm-${pkgname}.desktop"
  "policytool-${pkgname}.desktop"
)
sha256sums=(
  "9a5c32411a6a06e22b69c495b7975034409fa1652d03aeb8eb5b6f59fd4594e0"
  "9c64997edfce44e29296bfbd0cf90abf8b6b9ef2ea64733adae3bdac9ae2c5a6"
  "50d8f76c1205e437c290a3d3fe1274e2cddbcac4dde0a95300cc473200176154"
  "dfa19d10ea5614fda73cee31c592601e3a24c567150a6a7243dd7bcd1e3540cc"
  "44c97f46d2706f4410ebf838bb26e21e6dee3a7e0b357cf91d68c75cf9d1e011"
  "6bfccd461abd3ee2eba600c19f3f3cc5be95bff841e96ea1bc05b234fba9e27d"
)
DLAGENTS=("${DLAGENTS[@]//curl -/curl -b "oraclelicense=a" -}")

package() {
  _jvmdir="/usr/lib/jvm/java-${pkgver%u*}-${pkgname}"
  _workdir="${srcdir}/${pkgname%-bcl}1.${pkgver%u*}.0_${pkgver#*u}"

  set +u; msg2 "Creating directory structure..."; set -u
  install -dm755 "${pkgdir}/etc/.java/.systemPrefs"
  install -dm755 "${pkgdir}/usr/lib/jvm/java-${pkgver%u*}-${pkgname}/bin"
  install -dm755 "${pkgdir}/usr/lib/mozilla/plugins"
  install -dm755 "${pkgdir}/usr/share/licenses/java-${pkgname}"

  set +u; msg2 "Removing redundancies..."; set -u

  rm -rf "${_workdir}/jre/lib/desktop/icons/HighContrast/"
  rm -rf "${_workdir}/jre/lib/desktop/icons/HighContrastInverse/"
  rm -rf "${_workdir}/jre/lib/desktop/icons/LowContrast/"
  rm -rf "${_workdir}/jre/lib/fontconfig."*.bfc
  rm -rf "${_workdir}/jre/lib/fontconfig."*.properties.src
  rm -rf "${_workdir}/jre/"*.txt
  rm -rf "${_workdir}/jre/COPYRIGHT"
  rm -rf "${_workdir}/jre/LICENSE"
  rm -rf "${_workdir}/jre/README"
  rm -rf "${_workdir}/jre/plugin/"
  rm -rf "${_workdir}/man/ja"

  set +u; msg2 "Moving contents..."; set -u
  mv "${_workdir}/"* "${pkgdir}${_jvmdir}"

  set +u; msg2 "Replacing duplicate binaries in bin/ with links to jre/bin/ ..."; set -u
  local _i
  for _i in "${pkgdir}${_jvmdir}/jre/bin/"*; do
    ln -sf "${_jvmdir}/jre/bin/${_i##*/}" "${pkgdir}${_jvmdir}/bin/${_i##*/}"
  done

  set +u; msg2 "Adding ${pkgname} suffix to .desktop and icons (sun-java.png -> sun-java-${pkgname}.png) ..."; set -u
  local _i
  for _i in $(find "${pkgdir}${_jvmdir}/jre/lib/desktop/" -type "f"); do
    rename -- "." "-${pkgname}." "${_i}"
  done
  
  set +u; msg2 "Fixing .desktop paths..."; set -u
  sed -e "s|Exec=|Exec=${_jvmdir}/jre/bin/|" -i "${pkgdir}${_jvmdir}/jre/lib/desktop/applications/"*
  sed -e "s|.png|-${pkgname}.png|" -i "${pkgdir}${_jvmdir}/jre/lib/desktop/applications/"*

  set +u; msg2 "Moving .desktops and icons to /usr/share ..."; set -u
  mv "${pkgdir}${_jvmdir}/jre/lib/desktop/"* "${pkgdir}/usr/share/"
  install -m644 "${srcdir}/"*.desktop "${pkgdir}/usr/share/applications/"

  set +u; msg2 "Moving confs to /etc and link back to /usr: /usr/lib/jvm/java-${pkgname}/jre/lib -> /etc ..."; set -u
  local _new_etc_path
  for _new_etc_path in "${backup[@]}"; do
    # Old location
    local _old_usr_path="${pkgdir}${_jvmdir}/jre/lib/${_new_etc_path#*${pkgname}/}"

    # Move
    install -Dm644 "${_old_usr_path}" "${pkgdir}/${_new_etc_path}"
    ln -sf "/${_new_etc_path}" "${_old_usr_path}"
  done

  set +u; msg2 "Linking NPAPI plugin..."; set -u
  ln -s "${_jvmdir}/jre/lib/amd64/libnpjp2.so" "${pkgdir}/usr/lib/mozilla/plugins/libnpjp2-${pkgname}.so"

  set +u; msg2 "Replacing JKS keystore with ca-certificates-java ..."; set -u
  ln -sf "/etc/ssl/certs/java/cacerts" "${pkgdir}${_jvmdir}/jre/lib/security/cacerts"

  set +u; msg2 "Adding suffix to man pages..."; set -u
  for _i in $(find "${pkgdir}${_jvmdir}/man/" -type "f"); do
    rename -- ".1" "-${pkgname}.1" "${_i}"
  done

  set +u; msg2 "Moving man pages..."; set -u
  mv "${pkgdir}${_jvmdir}/man/ja_JP.UTF-8/" "${pkgdir}${_jvmdir}/man/ja"
  mv "${pkgdir}${_jvmdir}/man/" "${pkgdir}/usr/share"

  set +u; msg2 "Moving/Linking licenses..."; set -u
  mv "${pkgdir}${_jvmdir}/COPYRIGHT" "${pkgdir}/usr/share/licenses/java-${pkgname}/"
  mv "${pkgdir}${_jvmdir}/LICENSE" "${pkgdir}/usr/share/licenses/java-${pkgname}/"
  mv "${pkgdir}${_jvmdir}/"*.txt "${pkgdir}/usr/share/licenses/java-${pkgname}/"
  ln -s "/usr/share/licenses/java-${pkgname}/" "${pkgdir}/usr/share/licenses/${pkgname}"

  set +u; msg2 "Installing Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files..."; set -u
  # Replace default "strong", but limited, cryptography to get an "unlimited strength" one for
  # things like 256-bit AES. Enabled by default in OpenJDK:
  # - http://suhothayan.blogspot.com/2012/05/how-to-install-java-cryptography.html
  # - http://www.eyrie.org/~eagle/notes/debian/jce-policy.html
  install -m644 "${srcdir}/UnlimitedJCEPolicyJDK${pkgver%u*}/"*.jar "${pkgdir}${_jvmdir}/jre/lib/security/"
  install -Dm644 "${srcdir}/UnlimitedJCEPolicyJDK${pkgver%u*}/README.txt" \
  "${pkgdir}/usr/share/doc/${pkgname}/README_-_Java_JCE_Unlimited_Strength.txt"

  set +u; msg2 "Enabling copy+paste in unsigned applets..."; set -u
  # Copy/paste from system clipboard to unsigned Java applets has been disabled since 6u24:
  # - https://blogs.oracle.com/kyle/entry/copy_and_paste_in_java
  # - http://slightlyrandombrokenthoughts.blogspot.com/2011/03/oracle-java-applet-clipboard-injection.html
  local _text='\
        // (AUR) Allow unsigned applets to read system clipboard, see:
        // - https://blogs.oracle.com/kyle/entry/copy_and_paste_in_java
        // - http://slightlyrandombrokenthoughts.blogspot.com/2011/03/oracle-java-applet-clipboard-injection.html
        permission java.awt.AWTPermission "accessClipboard";'
  local _lf=$'\n'
  _text="${_text//${_lf}/\\n}"
  local _line
  _line="$(awk "/permission/{a=NR}; END{print a}" "${pkgdir}/etc/java-${pkgname}/security/java.policy")"
  sed -e "${_line} a ${_text}" -i "${pkgdir}/etc/java-${pkgname}/security/java.policy"
  set +u
}
