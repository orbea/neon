AC_PREREQ(2.58) dnl 2.58 required for AS_HELP_STRING

dnl Extract the version (sans LF) from .version, created at release-time.
m4_define(ne_version, [m4_translit(m4_include(.version), [
])])

AC_INIT(neon, ne_version,,,[https://notroj.github.io/neon/])

AC_COPYRIGHT([Copyright 2000-2022 Joe Orton and others
This configure script may be copied, distributed and modified under the 
terms of the GNU Library General Public license; see src/COPYING.LIB for
more details.])

AC_CONFIG_HEADERS([config.h])
AC_CONFIG_SRCDIR(src/ne_request.c)

NEON_WITH_LIBS

# Pass through initial LDFLAGS verbatim to neon-config, so that extra
# libraries which are detected (e.g. OpenSSL) can still be found when
# building using the --libs output of neon-config.
user_LDFLAGS=$LDFLAGS

# By default, allow 'make install' to work.
ALLOW_INSTALL=yes

AC_DEFINE([_GNU_SOURCE], 1, [Always defined to enable GNU extensions])
AC_DEFINE([NEON_IS_LIBRARY], 1, [Defined when neon is built as a library])

AH_BOTTOM([
/* Enable leak-tracking versions of ne_*alloc when NEON_MEMLEAK is enabled */
#ifdef NEON_MEMLEAK
# include "memleak.h"
#endif])

AC_PROG_INSTALL

AC_ARG_ENABLE(webdav, 
  AS_HELP_STRING([--disable-webdav],[disable WebDAV support]))

if test "$enable_webdav" = "no"; then
  NEON_WITHOUT_WEBDAV
else
  # Yes, we do need an XML parser. The _BUNDLED macros handle
  # this normally.
  NEON_NEED_XML_PARSER=yes
fi

# The bundled macros also set this, which makes sure we recurse
# into the 'src' directory.
NEON_BUILD_BUNDLED=yes

# Define NEON_VERSION etc and make the appropriate substitutions.
NE_VERSIONS_BUNDLED

LIBNEON_SOURCE_CHECKS

dnl Avoid libtool 1.5 bug where configure fails if a C++ compiler
dnl is not available.
m4_ifdef([AC_LIBTOOL_TAGS], [AC_LIBTOOL_TAGS([])])

AC_DISABLE_SHARED
m4_ifdef([LT_INIT], [LT_INIT], [AC_PROG_LIBTOOL])

AC_EXEEXT

# Checks to compile test suite
NEON_TEST

# Use the libtool-type build.
NEON_LIBTOOL_BUILD
# Find an XML parser
NEON_XML_PARSER
# Internationalization support.
NEON_I18N()

# Extra checks for debugging, compiler warnings
NEON_DEBUG
# Leave till last to prevent CFLAGS affecting checks.
NEON_WARNINGS

CPPFLAGS="$CPPFLAGS -I\${top_builddir}"

AC_ARG_ENABLE(memleak, 
  AS_HELP_STRING([--enable-memleak],
                 [for test builds only: enable memory leak checking]))

if test "$enable_memleak" = "yes"; then
  CPPFLAGS="$CPPFLAGS -DNEON_MEMLEAK -I\$(top_srcdir)/src"
  # disable 'make install'
  ALLOW_INSTALL=memleak
fi

# Enable tests for optional features
TESTS="\$(BASIC_TESTS)"
HELPERS=""
if test $NE_FLAG_SSL = yes; then
   # Only enable SSL tests if an openssl binary is found (needed to make
   # certs etc).
   AC_PATH_PROG(OPENSSL, openssl, notfound)
   if test "$OPENSSL" != "notfound"; then
     TESTS="$TESTS \$(SSL_TESTS)"
     HELPERS="$HELPERS \$(SSL_HELPERS)"
   else
     AC_MSG_WARN([no openssl binary in \$PATH: SSL tests disabled])
   fi

   AC_PATH_PROG(CERTUTIL, certutil, notfound)
   AC_PATH_PROG(PK12UTIL, pk12util, notfound)
fi
if test $NE_FLAG_ZLIB = yes; then
   TESTS="$TESTS \$(ZLIB_TESTS)"
   HELPERS="$HELPERS \$(ZLIB_HELPERS)"
fi
if test x$enable_webdav != xno; then
   TESTS="$TESTS \$(DAV_TESTS)"
fi

AC_ARG_ENABLE(tests-install,
  AS_HELP_STRING([--enable-tests-install],
                 [enable installation of the test suite]),,
  [enable_tests_install=no])

# If test suite installation is not required, it's more
# efficient to link the test programs using -no-install:
if test "$enable_tests_install" = "no"; then
  TEST_LDFLAGS="-no-install"
fi

AC_SUBST(TEST_LDFLAGS)

AC_PATH_PROG(PKG_CONFIG, pkg-config, no)
if test "$PKG_CONFIG" != "no"; then
   # pkg-config >= 0.18 will use "Libs.private" iff necessary,
   # older versions which don't recognize that field may always
   # need all libraries in Libs.
   if $PKG_CONFIG --atleast-pkgconfig-version=0.18; then :; else
      NEON_PC_LIBS=${NEON_LIBS}
   fi
fi
AC_SUBST(NEON_PC_LIBS)

# Pass the interface version on to libtool when linking libneon.la
NEON_LINK_FLAGS="-version-info ${NE_LIBTOOL_VERSINFO}"
# If any non-default ABI variations are used, add them to the SONAME:
### disabled for backwards-compat with 0.27.x
#if test "x${NE_LIBTOOL_RELEASE}y" != "xy"; then
#   NEON_LINK_FLAGS="${NEON_LINK_FLAGS} -release ${NE_LIBTOOL_RELEASE}"
#fi

gl_LD_VERSION_SCRIPT
# If ld version scripts are supported, enable symbol versioning.
# Otherwise, fall back to any libtool-supported symbol export
# restrictions; ne__* symbols are not exported.
if test "x$have_ld_version_script" = "xyes"; then
  NEON_LINK_FLAGS="$NEON_LINK_FLAGS -Wl,--version-script=\$(top_srcdir)/src/neon.vers"
else
  NEON_LINK_FLAGS="$NEON_LINK_FLAGS -export-symbols-regex '^ne_[[^_]]'"
fi 

if test x${enable_shared}${pic_mode}z = xnodefaultz; then
   CFLAGS="$CFLAGS -prefer-pic"
   AC_MSG_NOTICE([Using PIC for static library build])
fi

if test x${enable_shared} = xno; then
   # Defining NE_PRIVATE as the empty string would work; using a
   # non-empty but redundant string 'extern' avoids any possible cpp
   # confusion with from an empty macro.
   CPPFLAGS="$CPPFLAGS -DNE_PRIVATE=extern"
   AC_MSG_NOTICE([Private symbol suppression disabled for static library build])
fi

# Bundled language catalogs
ALL_LINGUAS="cs de fr ja ka nn pl ru tr zh_CN"
AC_SUBST(ALL_LINGUAS)

AC_CONFIG_FILES([neon-config], [chmod +x neon-config])
AC_CONFIG_FILES([Makefile src/Makefile test/Makefile neon.pc])
AC_CONFIG_FILES([test/makekeys:test/makekeys.sh], [chmod +x test/makekeys])

AC_SUBST(NEON_VERSION)
AC_SUBST(NEON_BUILD_BUNDLED)
AC_SUBST(top_builddir)
AC_SUBST(user_LDFLAGS)
AC_SUBST(HELPERS)
AC_SUBST(TESTS)
AC_SUBST(ALLOW_INSTALL)

AC_OUTPUT

# for VPATH builds:
test -d test/common || mkdir test/common

AC_MSG_NOTICE([Configured to build AC_PACKAGE_STRING:

  Install prefix:  ${prefix}
  Compiler:        ${CC}
  XML Parser:      ${neon_xml_parser_message}
  SSL library:     ${ne_SSL_message}
  zlib support:    ${ne_ZLIB_message}
  GSSAPI support:  ${ne_GSSAPI_message}
  Build libraries: Shared=${enable_shared}, Static=${enable_static}
])

case $ALLOW_INSTALL in
memleak)
  AC_MSG_NOTICE([Configured with development-only flags:

WARNING: This copy of neon has been configured with memory leak checking
WARNING: enabled, which should only be used in a development build of neon.
WARNING: This neon library should not be installed for use by applications.
]);;
esac
