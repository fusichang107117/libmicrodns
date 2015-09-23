m4_define([MDNS_MAJOR], [0])
m4_define([MDNS_MINOR], [0])
m4_define([MDNS_MICRO], [1])

AC_COPYRIGHT([Copyright 2014-2015 VideoLabs])

AC_INIT([microDNS], [MDNS_MAJOR.MDNS_MINOR.MDNS_MICRO])

AM_INIT_AUTOMAKE()
AC_CONFIG_HEADERS([config.h])

AC_PROG_LIBTOOL
AC_PROG_CC_C99
AC_C_INLINE

dnl
dnl  Debugging mode
dnl
AC_ARG_ENABLE(debug,
  [AS_HELP_STRING([--enable-debug],
    [build with run-time assertions (default disabled)])],,
  [enable_debug="no"])
AH_TEMPLATE(NDEBUG,
            [Define to 1 if debug code should NOT be compiled])
AS_IF([test "${enable_debug}" != "no"], [
  AC_CHECK_HEADERS([valgrind/valgrind.h])
], [
  AC_DEFINE(NDEBUG)
])

AC_CHECK_HEADERS([arpa/inet.h])
AC_CHECK_FUNCS([inet_ntop])

dnl
dnl  Check the operating system
dnl
HAVE_WIN64="0"
HAVE_IOS="0"
HAVE_OSX="0"
HAVE_TVOS="0"

LIBSOCKET=

case "${host_os}" in
  "")
    SYS=unknown
    ;;
  linux*)
    SYS=linux
    ;;
  bsdi*)
    SYS=bsdi
    ;;
  freebsd*)
    SYS=freebsd
    CPPFLAGS="${CPPFLAGS} -I/usr/local/include"
    LDFLAGS="${LDFLAGS} -L/usr/local/lib"
    ;;
  openbsd*)
    SYS=openbsd
    CPPFLAGS="${CPPFLAGS} -I/usr/local/include -I/usr/X11R6/include"
    LDFLAGS="${LDFLAGS} -L/usr/local/lib -L/usr/X11R6/lib/"
    ;;
  netbsd*)
    SYS=netbsd
    ;;
  dragonfly*)
    SYS=dragonfly
    ;;
  darwin*)

    dnl Force gcc "-arch" flag
    ARCH_flag=""
    case "${host}" in
      i?86*)
        ARCH_flag="-arch i386"
      ;;
      ppc64*)
        ARCH_flag="-arch ppc64"
      ;;
      ppc*)
        ARCH_flag="-arch ppc"
      ;;
      x86_64*)
        ARCH_flag="-arch x86_64"
      ;;
      arm*)
        ac_cv_c_bigendian="no"
        ac_cv_c_attribute_packed="no"
      ;;
    esac

    SYS=darwin
    CFLAGS="${CFLAGS} -D_INTL_REDIRECT_MACROS -std=gnu11 ${ARCH_flag}"
    CXX="${CXX} -stdlib=libc++ -std=c++11"
    CXXFLAGS="${CXXFLAGS} -D_INTL_REDIRECT_MACROS ${ARCH_flag}"
    CPPFLAGS="${CPPFLAGS} ${ARCH_flag}"
    OBJCFLAGS="${OBJCFLAGS} -D_INTL_REDIRECT_MACROS -std=gnu11 ${ARCH_flag}"
    LDFLAGS="${LDFLAGS} -Wl,-headerpad_max_install_names ${ARCH_flag}"

    AC_EGREP_CPP(yes,
            [#import <TargetConditionals.h>
             #if TARGET_OS_IPHONE
             yes
             #endif],
             [HAVE_IOS="1"],
             [HAVE_OSX="1"])

    AC_EGREP_CPP(yes,
            [#import <TargetConditionals.h>
             #if TARGET_OS_TV
             yes
             #endif],
             [HAVE_TVOS="1"
             HAVE_IOS="0"
             HAVE_OSX="0"
             ],)

    dnl Allow binaries created on Lion to run on earlier releases
    AC_EGREP_CPP(yes,
           [#import <Cocoa/Cocoa.h>
            #ifdef MAC_OS_X_VERSION_10_7
            yes
            #endif],
            [AC_MSG_RESULT([yes])
            AC_LIBOBJ([getdelim])
            AC_LIBOBJ([strndup])
            AC_LIBOBJ([strnlen])],)

    dnl
    dnl  Handle Mac OS X SDK flags
    dnl
    AC_ARG_WITH(macosx-sdk,
      [AS_HELP_STRING([--with-macosx-sdk=DIR],
        [compile using the SDK in DIR])])
    if test "${with_macosx_sdk}" != "" ; then
        test ! -d "${with_macosx_sdk}" && AC_MSG_ERROR([SDK "${with_macosx_sdk}" not found])
        CPP="${CPP} -isysroot ${with_macosx_sdk}"
        CC="${CC} -isysroot ${with_macosx_sdk}"
        CXX="${CXX} -isysroot ${with_macosx_sdk}"
        OBJC="${OBJC} -isysroot ${with_macosx_sdk}"
        LD="${LD} -syslibroot ${with_macosx_sdk}"
    fi
    AC_ARG_WITH(macosx-version-min,
      [AS_HELP_STRING([--with-macosx-version-min=VERSION],
        [compile for Mac OS X VERSION and above])])
    if test "${with_macosx_version_min}" != "" ; then
        CPP="${CPP} -mmacosx-version-min=${with_macosx_version_min}"
        CC="${CC} -mmacosx-version-min=${with_macosx_version_min}"
        CXX="${CXX} -mmacosx-version-min=${with_macosx_version_min}"
        OBJC="${OBJC} -mmacosx-version-min=${with_macosx_version_min}"
        LD="${LD} -mmacosx_version_min=${with_macosx_version_min}"
    fi
    ;;
  *mingw32* | *cygwin* | *wince* | *mingwce*)
    AC_DEFINE([_UNICODE], [1], [Define to 1 for Unicode (Wide Chars) APIs.])
    AC_DEFINE([UNICODE], [1], [Define to 1 for Unicode (Wide Chars) APIs.])
    AC_DEFINE([_POSIX_C_SOURCE], [200809L], [IEEE Std 1003.1.])
    AC_DEFINE([_BSD_SOURCE], [1], [ISO C, POSIX, and 4.3BSD things.])

    case "${host_os}" in
      *mingw32*)
        SYS=mingw32
        ;;
      *cygwin*)
        SYS=cygwin
        ;;
    esac

    if test "${SYS}" = "mingw32"; then
        # DEP, ASLR, NO SEH
        LDFLAGS="${LDFLAGS} -Wl,--nxcompat -Wl,--no-seh -Wl,--dynamicbase"

        AC_CHECK_PROGS(U2D, [unix2dos todos], unix2dos)
        ac_default_prefix="`pwd`/_win32"
        DESTDIR="`pwd`/_win32/"

        dnl
        dnl NSIS/MSI Installer prefix and WIN64
        dnl
        case "${host}" in
            amd64*|x86_64*)
                HAVE_WIN64="1"
                WINDOWS_ARCH="x64"
                PROGRAMFILES="PROGRAMFILES64"
            ;;
            *)
                WINDOWS_ARCH="x86"
                PROGRAMFILES="PROGRAMFILES"
            ;;
        esac
	LIBSOCKET='-lws2_32'
        AC_SUBST(WINDOWS_ARCH)
        AC_SUBST(PROGRAMFILES)

    fi
    ;;
  *nto*)
    SYS=nto
    ;;
  solaris*)
    SYS=solaris
    ;;
  hpux*)
    SYS=hpux
    ;;
  *os2*)
    SYS=os2
    LDFLAGS="${LDFLAGS} -Zomf -Zbin-files -Zargs-wild -Zhigh-mem"
    AC_LIBOBJ([freeaddrinfo])
    ;;
  *)
    SYS="${host_os}"
    ;;
esac
AM_CONDITIONAL(HAVE_WIN32,   test "${SYS}" = "mingw32")

AC_SUBST([LIBSOCKET])

PKG_INSTALLDIR

AC_CONFIG_FILES([
Makefile
microdns.pc
])

AC_OUTPUT