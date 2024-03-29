# configure.ac                                          -*- Autoconf -*-
# Copyright (C) 2011, 2012 g10 Code GmbH
#
# This file is part of nPth.
#
# nPth is free software; you can redistribute it and/or modify it
# under the terms of the GNU Lesser General Public License as
# published by the Free Software Foundation; either version 2.1 of the
# License, or (at your option) any later version.
#
# nPth is distributed in the hope that it will be useful, but WITHOUT
# ANY WARRANTY; without even the implied warranty of MERCHANTABILITY
# or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU Lesser General
# Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public
# License along with this program; if not, see <http://www.gnu.org/licenses/>.

# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
min_automake_version="1.14"

# To build a release you need to create a tag with the version number
# (git tag -s npth-n.m) and run "./autogen.sh --force".  Please bump
# the version number immediately after the release and do another
# commit and push so that the git magic is able to work.
# See below for the LT versions.
m4_define([mym4_package],[npth])
m4_define([mym4_major], [1])
m4_define([mym4_minor], [8])

# To start a new development series, i.e a new major or minor number
# you need to mark an arbitrary commit before the first beta release
# with an annotated tag.  For example a 2.1 branch starts off with
# the tag "foo-2.1-base".  This is used as the base for counting
# beta numbers before the first release of a series.

# Below is m4 magic to extract and compute the git revision number,
# the decimalized short revision number, a beta version string and a
# flag indicating a development version (mym4_isbeta).  Note that the
# m4 processing is done by autoconf and not during the configure run.
m4_define([mym4_verslist], m4_split(m4_esyscmd([./autogen.sh --find-version] \
                           mym4_package mym4_major mym4_minor),[:]))
m4_define([mym4_isbeta],       m4_argn(2, mym4_verslist))
m4_define([mym4_version],      m4_argn(4, mym4_verslist))
m4_define([mym4_revision],     m4_argn(7, mym4_verslist))
m4_define([mym4_revision_dec], m4_argn(8, mym4_verslist))
m4_esyscmd([echo ]mym4_version[>VERSION])
AC_INIT([mym4_package],[mym4_version],[https://bugs.gnupg.org])
# LT Version numbers, remember to change them just *before* a release.
#   (Code changed:			REVISION++)
#   (Interfaces added/removed/changed:	CURRENT++, REVISION=0)
#   (Interfaces added:			AGE++)
#   (Interfaces removed:		AGE=0)
#
LIBNPTH_LT_CURRENT=2
LIBNPTH_LT_AGE=2
LIBNPTH_LT_REVISION=0
################################################

AC_SUBST(LIBNPTH_LT_CURRENT)
AC_SUBST(LIBNPTH_LT_AGE)
AC_SUBST(LIBNPTH_LT_REVISION)

VERSION_NUMBER=m4_esyscmd(printf "0x%02x%02x00" mym4_major mym4_minor)
AC_SUBST(VERSION_NUMBER)

# If the API is changed in an incompatible way: increment the next counter.
NPTH_CONFIG_API_VERSION=1
##############################################

AC_CONFIG_AUX_DIR([build-aux])
AM_INIT_AUTOMAKE([serial-tests dist-bzip2 no-dist-gzip])
AM_MAINTAINER_MODE
AC_CONFIG_SRCDIR([src/npth.c])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])
AC_CANONICAL_HOST
AM_SILENT_RULES

# Enable GNU extensions on systems that have them.
AC_USE_SYSTEM_EXTENSIONS

# Taken from mpfr-4.0.1, then modified for LDADD_FOR_TESTS_KLUDGE
dnl Under Linux, make sure that the old dtags are used if LD_LIBRARY_PATH
dnl is defined. The issue is that with the new dtags, LD_LIBRARY_PATH has
dnl the precedence over the run path, so that if a compatible MPFR library
dnl is installed in some directory from $LD_LIBRARY_PATH, then the tested
dnl MPFR library will be this library instead of the MPFR library from the
dnl build tree. Other OS with the same issue might be added later.
dnl
dnl References:
dnl   https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=859732
dnl   http://lists.gnu.org/archive/html/libtool/2017-05/msg00000.html
dnl
dnl We need to check whether --disable-new-dtags is supported as alternate
dnl linkers may be used (e.g., with tcc: CC=tcc LD=tcc).
dnl
case $host in
  *-*-linux*)
    if test -n "$LD_LIBRARY_PATH"; then
      saved_LDFLAGS="$LDFLAGS"
      LDADD_FOR_TESTS_KLUDGE="-Wl,--disable-new-dtags"
      LDFLAGS="$LDFLAGS $LDADD_FOR_TESTS_KLUDGE"
      AC_MSG_CHECKING(whether --disable-new-dtags is supported by the linker)
      AC_LINK_IFELSE([AC_LANG_SOURCE([[
int main (void) { return 0; }
      ]])],
      [AC_MSG_RESULT(yes (use it since LD_LIBRARY_PATH is set))],
      [AC_MSG_RESULT(no)
       LDADD_FOR_TESTS_KLUDGE=""
      ])
      LDFLAGS="$saved_LDFLAGS"
    fi
    ;;
esac
AC_SUBST([LDADD_FOR_TESTS_KLUDGE])

AH_VERBATIM([_REENTRANT],
[#ifndef _REENTRANT
# define _REENTRANT 1
#endif])

# Checks for programs.
AC_PROG_CC

if test "$GCC" = yes; then
    CFLAGS="$CFLAGS -Wall -Wcast-align -Wshadow -Wstrict-prototypes"

    AC_MSG_CHECKING([if gcc supports -Wpointer-arith])
    _gcc_cflags_save=$CFLAGS
    CFLAGS="-Wpointer-arith"
    AC_COMPILE_IFELSE([AC_LANG_PROGRAM([])],_gcc_wopt=yes,_gcc_wopt=no)
    AC_MSG_RESULT($_gcc_wopt)
    CFLAGS=$_gcc_cflags_save;
    if test x"$_gcc_wopt" = xyes ; then
       CFLAGS="$CFLAGS -Wpointer-arith"
    fi
fi

# We used to provide npth-config command always.  Now, it's
# gpgrt-config command with npth.pc configuration file, which does
# same thing.
AC_ARG_ENABLE(install-npth-config,
  AS_HELP_STRING([--enable-install-npth-config],[install npth-config command]),
  install_npth_config=$enableval)
AM_CONDITIONAL(INSTALL_NPTH_CONFIG, test "$install_npth_config" = "yes")


run_tests="yes"
AC_ARG_ENABLE(tests,
  AS_HELP_STRING([--disable-tests],[disable tests]),
         run_tests=$enableval)
AM_CONDITIONAL(RUN_TESTS, test "$run_tests" = "yes")


# Don't default to build static libs.
LT_PREREQ([2.2.6])
LT_INIT([win32-dll disable-static])
LT_LANG([Windows Resource])


# For now we hardcode the use of version scripts.  It would be better
# to write a test for this or even implement this within libtool.
have_ld_version_script=no
have_fork_unsafe_semaphore=no
case "${host}" in
    *-*-linux*)
        have_ld_version_script=yes
        ;;
    *-*-gnu*)
        have_ld_version_script=yes
        ;;
    *-apple-darwin*)
        AC_SEARCH_LIBS([dispatch_semaphore_create],[dispatch],
        [AC_DEFINE([HAVE_LIB_DISPATCH],1,[Defined if we have libdispatch])])
        if test "$ac_cv_search_dispatch_semaphore_create" = no; then
           AC_DEFINE(HAVE_NO_POSIX_SEMAPHORE,1,[Define if no good semaphore.])
        fi
        ;;
    *-*-aix*)
        have_fork_unsafe_semaphore=yes
        ;;
    *-*-dragonfly*|*-*-freebsd*|*-*-netbsd*|*-*-hpux*)
        LIBS="-lpthread $LIBS"
        AC_SEARCH_LIBS([sem_init],[rt])
        ;;
esac

AM_CONDITIONAL(HAVE_LD_VERSION_SCRIPT, test "$have_ld_version_script" = "yes")

#
# Specify how we support our local modification of libtool for Windows
# 64-bit.  Options are:
#
# (1) apply: when appying patch fails, it results failure of entire build
# (2) never: never apply the patch (no try)
# (3) try: use patched if it goes well, use original if fails
#
AC_ARG_WITH([libtool-modification],
  AS_HELP_STRING([--with-libtool-modification=apply|never|try],
                 [how to handle libtool modification (default=never)]),
                 build_libtool_modification=$withval,
                 build_libtool_modification=never)

#
# Apply a patch (locally maintained one of ours) to libtool
#
case $host in
  x86_64-*mingw32*)
AC_CONFIG_COMMANDS([libtool-patch],[[
  if test "$build_selection" = never; then
    echo "patch not applied"
  elif (mv -f libtool libtool.orig; \
        sed -f $srcdir/build-aux/libtool-patch.sed libtool.orig >libtool); then
    echo "applied successfully"
  elif test "$build_selection" = try; then
    mv -f libtool.orig libtool
    echo "patch failed, thus, using original"
  else
    echo "patch failed"
    as_fn_exit 1
  fi
]],[build_selection=$build_libtool_modification])
  ;;
  *)
  ;;
esac

AM_CONDITIONAL(NPTH_FORK_FAILURE, test "$have_fork_unsafe_semaphore" = "yes")


# Set some default values
config_libs="-lnpth $LIBS"
have_w32_system=no
have_w32ce_system=no
have_w64_system=no

# Define OS macros
case "${host}" in
    x86_64-*mingw32*)
        have_w64_system=yes
        ;;
    *-mingw32ce*)
        have_w32ce_system=yes
        ;;
esac
case "${host}" in
    *-mingw32ce*|*-mingw32*)
        have_w32_system=yes
        ;;
    *)
        ;;
esac

if test "$have_w32_system" = yes; then
   AC_DEFINE(HAVE_W32_SYSTEM,1,
             [Defined if we run on any kind of W32 API based system])
fi
AM_CONDITIONAL(HAVE_W32_SYSTEM, test "$have_w32_system" = yes)

if test "$have_w64_system" = yes; then
   AC_DEFINE(HAVE_W64_SYSTEM,1,
             [Defined if we run on a 64 bit W32 API based system])
fi
AM_CONDITIONAL(HAVE_W64_SYSTEM, test "$have_w64_system" = yes)

#
# Generate values for the DLL version info
#
if test "$have_w32_system" = yes; then
    BUILD_ISODATE=`date --iso-8601`
    changequote(,)dnl
    BUILD_FILEVERSION=`echo "$VERSION" | sed 's/\([0-9.]*\).*/\1./;s/\./,/g'`
    changequote([,])dnl
    BUILD_FILEVERSION="${BUILD_FILEVERSION}git_revision_dec"
fi
AC_SUBST(BUILD_ISODATE)
AC_SUBST(BUILD_FILEVERSION)
AC_ARG_ENABLE([build-timestamp],
  AS_HELP_STRING([--enable-build-timestamp],[set an explicit build timestamp for reproducibility.
                  (default is the current time in ISO-8601 format)]),
     [if test "$enableval" = "yes"; then
        BUILD_TIMESTAMP=`date -u +%Y-%m-%dT%H:%M+0000 2>/dev/null || date`
      else
        BUILD_TIMESTAMP="$enableval"
      fi],
     [BUILD_TIMESTAMP="<none>"])
AC_SUBST(BUILD_TIMESTAMP)
AC_DEFINE_UNQUOTED(BUILD_TIMESTAMP, "$BUILD_TIMESTAMP",
                   [The time this package was configured for a build])


#
# Checks for header files.
#
# fixme: For what do we need the sys/socket test?
AC_CHECK_HEADERS([sys/socket.h sys/select.h unistd.h sys/time.h time.h \
                  signal.h poll.h])
INSERT_SYS_SELECT_H=
if test x"$ac_cv_header_sys_select_h" = xyes; then
  INSERT_SYS_SELECT_H="include <sys/select.h>"
fi
AC_SUBST(INSERT_SYS_SELECT_H)

if test x"$ac_cv_header_sys_time_h" = xyes; then
  INSERT_SYS_TIME_H="include <sys/time.h>"
fi
AC_SUBST(INSERT_SYS_TIME_H)

if test x"$ac_cv_header_time_h" = xyes; then
  INSERT_TIME_H="include <time.h>"
fi
AC_SUBST(INSERT_TIME_H)

if test x"$ac_cv_header_signal_h" = xyes; then
  INSERT_SIGNAL_H="include <signal.h>"
fi
AC_SUBST(INSERT_SIGNAL_H)


# Some systems lack socklen_t - provide a replacement.
gl_TYPE_SOCKLEN_T
case "${host}" in
  *-*-mingw32*)
    # socklen_t may or may not be defined depending on what headers
    # are included.  To be safe we use int as this is the actual type.
    INSERT_SOCKLEN_T="define _npth_socklen_t int"
    ;;
  *)
    if test ".$gl_cv_socklen_t_equiv" = "."; then
      INSERT_SOCKLEN_T="define _npth_socklen_t socklen_t"
    else
      INSERT_SOCKLEN_T="define _npth_socklen_t ${gl_cv_socklen_t_equiv}"
    fi
esac
AC_SUBST(INSERT_SOCKLEN_T)


#
# Checks for typedefs, structures, and compiler characteristics.
#
AC_TYPE_PID_T
AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T


#
# Checks for libraries and functions.
#
#   We test for pthread_cancel because in glibc 2.34, libc includes
#   pthread_create and pthread_detach, but not pthread_cancel.
if test "$have_w32_system" = no; then
  AC_SEARCH_LIBS([pthread_cancel],[pthread])
  case "x$ac_cv_search_pthread_cancel" in
    xno)
      have_pthread=no
      ;;
    xnone\ required)
      have_pthread=yes
      ;;
    *)
      have_pthread=yes
      config_libs="$config_libs $ac_cv_search_pthread_cancel"
      ;;
  esac
  if test "$have_pthread" != no; then
    AC_DEFINE(HAVE_PTHREAD,1,[Define if we have pthread.])
    AC_CHECK_TYPE([pthread_rwlock_t])
    AC_CHECK_FUNCS([pthread_tryjoin_np pthread_setname_np pthread_getname_np])
    AC_CHECK_FUNCS([pthread_mutex_timedlock])
    AC_CHECK_FUNCS([pthread_rwlock_rdlock pthread_rwlock_wrlock])
    AC_CHECK_FUNCS([pthread_rwlock_timedrdlock pthread_rwlock_timedwrlock])
    AC_CHECK_FUNCS([pthread_rwlock_tryrdlock pthread_rwlock_trywrlock])
    AC_CHECK_FUNCS([pthread_atfork])
  fi
fi

INSERT_NO_RWLOCK="undef _NPTH_NO_RWLOCK"
if test x"$ac_cv_type_pthread_rwlock_t" = xno; then
  INSERT_NO_RWLOCK="define _NPTH_NO_RWLOCK 1"
fi
AC_SUBST(INSERT_NO_RWLOCK)

case "${host}" in
    *-*-linux-musl*)
        INSERT_EXPOSE_RWLOCK_API="1"
        ;;
    *-*-linux-gnu*|*-*-gnu*)
        INSERT_EXPOSE_RWLOCK_API="defined(__USE_UNIX98) || defined(__USE_XOPEN2K)"
        ;;
    *)
        INSERT_EXPOSE_RWLOCK_API="1"
        ;;
esac
AC_SUBST(INSERT_EXPOSE_RWLOCK_API)

AC_CHECK_FUNCS([select pselect gettimeofday ppoll])

npth_LIBSOCKET
config_libs="$config_libs $LIBSOCKET"

# Save and restore LIBS so e.g., -lrt, isn't added to it.  Otherwise, *all*
# programs in the package would end up linked with that potentially-shared
# library, inducing unnecessary run-time overhead.
LIB_CLOCK_GETTIME=
AC_SUBST([LIB_CLOCK_GETTIME])
gl_saved_libs=$LIBS
AC_SEARCH_LIBS([clock_gettime], [rt posix4],
               [if test "$ac_cv_search_clock_gettime" != "none required"; then
                  LIB_CLOCK_GETTIME=$ac_cv_search_clock_gettime
                  config_libs="$config_libs $LIB_CLOCK_GETTIME"
                fi
                AC_DEFINE([HAVE_CLOCK_GETTIME],1,
                  [Define to 1 if you have the `clock_gettime' function.])
                ])
LIBS=$gl_saved_libs


#
# Set NETLIBS
#
if test "$have_w32ce_system" = yes; then
   NETLIBS="-lws2 $NETLIBS"
elif test "$have_w32_system" = yes; then
   NETLIBS="-lws2_32 $NETLIBS"
fi
AC_SUBST(NETLIBS)


#
# Substitutions to set generated files in a Emacs buffer to read-only.
#
AC_SUBST(emacs_local_vars_begin, [['Local][ ][Variables:']])
AC_SUBST(emacs_local_vars_read_only, ['buffer-read-only: t'])
AC_SUBST(emacs_local_vars_end, ['End:'])


#
# Substitution used for npth-config
#
NPTH_CONFIG_LIBS="$config_libs"
NPTH_CONFIG_CFLAGS=""
NPTH_CONFIG_HOST="$host"
AC_SUBST(NPTH_CONFIG_API_VERSION)
AC_SUBST(NPTH_CONFIG_LIBS)
AC_SUBST(NPTH_CONFIG_CFLAGS)
AC_SUBST(NPTH_CONFIG_HOST)

#
# Last check.
#
die=no
if test "$have_w32_system" = no; then
  if test "$have_pthread" = "no"; then
     die=yes
     AC_MSG_NOTICE([[
***
*** You need Pthread to build this program.
*** Normally, this library comes with your system.  On Windows, you can use:
***   http://sourceware.org/pthreads-win32/
***]])
  fi
fi

if test "$die" = "yes"; then
    AC_MSG_ERROR([[
***
*** Required libraries not found. Please consult the above messages
*** and install them before running configure again.
***]])
fi

#
# Write output
#
AC_CONFIG_FILES([Makefile
npth.pc
src/npth.h
src/Makefile
w32/Makefile
tests/Makefile])
AC_CONFIG_FILES(npth-config, chmod +x npth-config)
AC_OUTPUT

echo "
        $PACKAGE_NAME v$PACKAGE_VERSION prepared for make

        Revision: mym4_revision  (mym4_revision_dec)
        Platform: $host
"
