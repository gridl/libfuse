AC_INIT(fuse, 3.1.0)
AC_PREREQ(2.59d)
AC_CONFIG_MACRO_DIR([m4])
AC_CANONICAL_TARGET
AM_INIT_AUTOMAKE([subdir-objects foreign])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES(yes)])
AC_CONFIG_HEADERS(include/config.h)

AC_PROG_LIBTOOL
AC_PROG_CC_STDC
AC_SYS_LARGEFILE
AC_PROG_MKDIR_P
AM_PROG_CC_C_O

case $target_os in
     *linux*)	arch=linux;;
     *netbsd*)  arch=netbsd;;
     *bsd*)	arch=bsd;;
     *)		arch=unknown;;
esac

if test "$ac_env_CFLAGS_set" != set; then
	CFLAGS="-Wall -W -Wno-sign-compare -Wstrict-prototypes -Wmissing-declarations -Wwrite-strings -g -O2 -fno-strict-aliasing"
fi

AC_ARG_ENABLE(lib,
	[  --enable-lib            Compile with library ])
AC_ARG_ENABLE(util,
	[  --enable-util           Compile with util ])
AC_ARG_ENABLE(example,
	[  --enable-example        Compile with examples ])
AC_ARG_ENABLE(test,
	[  --enable-test           Compile with tests ])
AC_ARG_ENABLE(mtab,
	[  --disable-mtab          Disable and ignore usage of /etc/mtab ])

AC_ARG_WITH(pkgconfigdir,
	    [  --with-pkgconfigdir=DIR      pkgconfig file in DIR @<:@LIBDIR/pkgconfig@:>@],
	    [pkgconfigdir=$withval],
	    [pkgconfigdir='${libdir}/pkgconfig'])
AC_SUBST(pkgconfigdir)

subdirs2="include"

if test "$enable_lib" != "no"; then
	subdirs2="$subdirs2 lib";
fi
if test "$arch" = linux -a "$enable_util" != "no"; then
	subdirs2="$subdirs2 util";
fi
if test "$enable_example" != "no"; then
	subdirs2="$subdirs2 example";
fi
if test "$enable_test" != "no"; then
	subdirs2="$subdirs2 test";
fi
if test "$enable_mtab" = "no" -o \
	"$arch" = "netbsd"; then
	AC_DEFINE(IGNORE_MTAB, 1, [Don't update /etc/mtab])
fi

AC_CHECK_FUNCS([fork setxattr fdatasync splice vmsplice utimensat pipe2])
AC_CHECK_FUNCS([posix_fallocate fstatat openat readlinkat])
AC_CHECK_MEMBERS([struct stat.st_atim])
AC_CHECK_MEMBERS([struct stat.st_atimespec])

LIBS=
AC_SEARCH_LIBS(dlopen, [dl])
AC_SEARCH_LIBS(clock_gettime, [rt])
libfuse_libs=$LIBS
LIBS=
AC_CHECK_LIB(ulockmgr, ulockmgr_op)
passthrough_fh_libs=$LIBS
AC_SUBST(passthrough_fh_libs)
LIBS=

AC_ARG_WITH([libiconv-prefix],
[  --with-libiconv-prefix=DIR  search for libiconv in DIR/include and DIR/lib], [
    for dir in `echo "$withval" | tr : ' '`; do
      if test -d $dir/include; then CPPFLAGS="$CPPFLAGS -I$dir/include"; fi
      if test -d $dir/lib; then LDFLAGS="$LDFLAGS -L$dir/lib"; fi
    done
   ])
AM_ICONV
libfuse_libs="$libfuse_libs $LTLIBICONV"
AM_CONDITIONAL(ICONV, test "$am_cv_func_iconv" = yes)
AC_SUBST(libfuse_libs)

if test -z "$MOUNT_FUSE_PATH"; then
	MOUNT_FUSE_PATH='${sbindir}'
	AC_MSG_NOTICE([MOUNT_FUSE_PATH env var not set, using default $MOUNT_FUSE_PATH])
fi
AC_SUBST(MOUNT_FUSE_PATH)
if test -z "$UDEV_RULES_PATH"; then
	UDEV_RULES_PATH='${libdir}/udev/rules.d'
	AC_MSG_NOTICE([UDEV_RULES_PATH env var not set, using default $UDEV_RULES_PATH])
fi
AC_SUBST(UDEV_RULES_PATH)
if test -z "$INIT_D_PATH"; then
	INIT_D_PATH='${sysconfdir}/init.d'
	AC_MSG_NOTICE([INIT_D_PATH env var not set, using default $INIT_D_PATH])
fi
AC_SUBST(INIT_D_PATH)

AC_SUBST(subdirs2)

AM_CONDITIONAL(LINUX, test "$arch" = linux)
AM_CONDITIONAL(NETBSD, test "$arch" = netbsd)
AM_CONDITIONAL(BSD, test "$arch" = bsd)

util_linux_ok=yes
if test "$arch" = linux -a "$cross_compiling" != "yes"; then
        AC_MSG_CHECKING([if umount supports --fake --no-canonicalize])
        if umount --help 2>&1 | grep -q -- "--fake" &&
           umount --help 2>&1 | grep -q -- "--no-canonicalize"; then
                AC_MSG_RESULT([yes])
        else
                AC_MSG_RESULT([no])
                util_linux_ok=no
        fi
fi

AC_CONFIG_FILES([fuse3.pc Makefile lib/Makefile util/Makefile example/Makefile include/Makefile doc/Makefile test/Makefile])
AC_OUTPUT

if test "$util_linux_ok" = no; then
	AC_MSG_WARN([
******************************************************************
* Please install util-linux version 2.18 or later which supports *
* --fake and --no-canonicalize options in mount and umount       *
******************************************************************])
fi
