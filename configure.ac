dnl -*- Mode: Autoconf; tab-width: 4; indent-tabs-mode: nil; fill-column: 102 -*-
dnl configure.ac serves as input for the GNU autoconf package
dnl in order to create a configure script.

AC_INIT([Hello],[1.0],[],[],[http://www.example.com/])

AC_PREREQ([2.59])

if test -n "$BUILD_TYPE"; then
    AC_MSG_ERROR([You have sourced config_host.mk in this shell.  This may lead to trouble, please run in a fresh (login) shell.])
fi

save_CC=$CC
save_CXX=$CXX

BUILD_TYPE="LibO"
SCPDEFS=""
GIT_NEEDED_SUBMODULES=""
LO_PATH= # used by path_munge to construct a PATH variable

PathFormat()
{
    formatted_path="$1"
    if test "$build_os" = "cygwin"; then
        pf_conv_to_dos=
        # spaces,parentheses,brackets,braces are problematic in pathname
        # so are backslashes
        case "$formatted_path" in
            *\ * | *\)* | *\(* | *\{* | *\}* | *\[* | *\]* | *\\* )
                pf_conv_to_dos="yes"
            ;;
        esac
        if test "$pf_conv_to_dos" = "yes"; then
            if test "$GNUMAKE_WIN_NATIVE" = "TRUE" ; then
                formatted_path=`cygpath -sm "$formatted_path"`
            else
                formatted_path=`cygpath -d "$formatted_path"`
            fi
            if test $? -ne 0;  then
                AC_MSG_ERROR([path conversion failed for "$1".])
            fi
        fi
        fp_count_colon=`echo "$formatted_path" | $GREP -c "[:]"`
        fp_count_slash=`echo "$formatted_path" | $GREP -c "[/]"`
        if test "$fp_count_slash$fp_count_colon" != "00"; then
            if test "$fp_count_colon" = "0"; then
                new_formatted_path=`realpath "$formatted_path"`
                if test $? -ne 0;  then
                    AC_MSG_WARN([realpath failed for "$1", not necessarily a problem.])
                else
                    formatted_path="$new_formatted_path"
                fi
            fi
            formatted_path=`cygpath -m "$formatted_path"`
            if test $? -ne 0;  then
                AC_MSG_ERROR([path conversion failed for "$1".])
            fi
        fi
    fi
}

AbsolutePath()
{
    # There appears to be no simple and portable method to get an absolute and
    # canonical path, so we try creating the directory if does not exist and
    # utilizing the shell and pwd.
    rel="$1"
    absolute_path=""
    test ! -e "$rel" && mkdir -p "$rel"
    if test -d "$rel" ; then
        cd "$rel" || AC_MSG_ERROR([absolute path resolution failed for "$rel".])
        absolute_path="$(pwd)"
        cd - > /dev/null
    else
        AC_MSG_ERROR([Failed to resolve absolute path.  "$rel" does not exist or is not a directory.])
    fi
}

rm -f warn
have_WARNINGS="no"
add_warning()
{
    if test "$have_WARNINGS" = "no"; then
        echo "*************************************" > warn
        have_WARNINGS="yes"
        if which tput >/dev/null 2>/dev/null && test `tput colors` -ge 8; then
            dnl <esc> as actual byte (U+1b), [ escaped using quadrigraph @<:@
            COLORWARN='*@<:@1;33;40m WARNING @<:@0m:'
        else
            COLORWARN="* WARNING :"
        fi
    fi
    echo "$COLORWARN $@" >> warn
}

dnl ===================================================================
dnl checks build and host OSes
dnl do this before argument processing to allow for platform dependent defaults
dnl ===================================================================
AC_CANONICAL_HOST

dnl ===================================================================
dnl Our version is defined by the AC_INIT() at the top of this script.
dnl ===================================================================

set `echo AC_PACKAGE_VERSION | sed "s/\./ /g"`

LIBO_VERSION_MAJOR=$1
LIBO_VERSION_MINOR=$2
LIBO_VERSION_MICRO=$3
LIBO_VERSION_PATCH=$4

LIBO_VERSION_SUFFIX=$5
# Split out LIBO_VERSION_SUFFIX_SUFFIX... horrible crack. But apparently wanted separately in
# openoffice.lst as ABOUTBOXPRODUCTVERSIONSUFFIX. Note that the double brackets are for m4's sake,
# they get undoubled before actually passed to sed.
LIBO_VERSION_SUFFIX_SUFFIX=`echo "$LIBO_VERSION_SUFFIX" | sed -e 's/.*[[a-zA-Z0-9]]\([[^a-zA-Z0-9]]*\)$/\1/'`
test -n "$LIBO_VERSION_SUFFIX_SUFFIX" && LIBO_VERSION_SUFFIX="${LIBO_VERSION_SUFFIX%${LIBO_VERSION_SUFFIX_SUFFIX}}"
# LIBO_VERSION_SUFFIX, if non-empty, should include the period separator
test -n "$LIBO_VERSION_SUFFIX" && LIBO_VERSION_SUFFIX=".$LIBO_VERSION_SUFFIX"

AC_SUBST(LIBO_VERSION_MAJOR)
AC_SUBST(LIBO_VERSION_MINOR)
AC_SUBST(LIBO_VERSION_MICRO)
AC_SUBST(LIBO_VERSION_PATCH)
AC_SUBST(MACOSX_BUNDLE_SHORTVERSION)
AC_SUBST(LIBO_VERSION_SUFFIX)
AC_SUBST(LIBO_VERSION_SUFFIX_SUFFIX)

AC_DEFINE_UNQUOTED(LIBO_VERSION_MAJOR,$LIBO_VERSION_MAJOR)
AC_DEFINE_UNQUOTED(LIBO_VERSION_MINOR,$LIBO_VERSION_MINOR)
AC_DEFINE_UNQUOTED(LIBO_VERSION_MICRO,$LIBO_VERSION_MICRO)
AC_DEFINE_UNQUOTED(LIBO_VERSION_PATCH,$LIBO_VERSION_PATCH)

LIBO_THIS_YEAR=`date +%Y`
AC_DEFINE_UNQUOTED(LIBO_THIS_YEAR,$LIBO_THIS_YEAR)

AC_PROG_EGREP
# AC_PROG_EGREP doesn't set GREP on all systems as well
AC_PATH_PROG(GREP, grep)

BUILDDIR=`pwd`
cd $srcdir
SRC_ROOT=`pwd`
cd $BUILDDIR
x_Cygwin=[\#]

dnl ===================================================================
dnl Search all the common names for GNU Make
dnl ===================================================================
AC_MSG_CHECKING([for GNU Make])

# try to use our own make if it is available and GNUMAKE was not already defined
if test -z "$GNUMAKE"; then
    if test -n "$LODE_HOME" -a -x "$LODE_HOME/opt/bin/make" ; then
        GNUMAKE="$LODE_HOME/opt/bin/make"
    elif test -x "/opt/lo/bin/make"; then
        GNUMAKE="/opt/lo/bin/make"
    fi
fi

GNUMAKE_WIN_NATIVE=
for a in "$MAKE" "$GNUMAKE" make gmake gnumake; do
    if test -n "$a"; then
        $a --version 2> /dev/null | grep GNU  2>&1 > /dev/null
        if test $? -eq 0;  then
            if test "$build_os" = "cygwin"; then
                if test -n "$($a -v | grep 'Built for Windows')" ; then
                    GNUMAKE="$(cygpath -m "$(which "$(cygpath -u $a)")")"
                    GNUMAKE_WIN_NATIVE="TRUE"
                else
                    GNUMAKE=`which $a`
                fi
            else
                GNUMAKE=`which $a`
            fi
            break
        fi
    fi
done
AC_MSG_RESULT($GNUMAKE)
if test -z "$GNUMAKE"; then
    AC_MSG_ERROR([not found. install GNU Make.])
else
    if test "$GNUMAKE_WIN_NATIVE" = "TRUE" ; then
        AC_MSG_NOTICE([Using a native Win32 GNU Make version.])
    fi
fi

win_short_path_for_make()
{
    local_short_path="$1"
    if test "$GNUMAKE_WIN_NATIVE" = "TRUE" ; then
        cygpath -sm "$local_short_path"
    else
        cygpath -u "$(cygpath -d "$local_short_path")"
    fi
}


if test "$build_os" = "cygwin"; then
    PathFormat "$SRC_ROOT"
    SRC_ROOT="$formatted_path"
    PathFormat "$BUILDDIR"
    BUILDDIR="$formatted_path"
    x_Cygwin=
fi

AC_SUBST(SRC_ROOT)
AC_SUBST(BUILDDIR)
AC_SUBST(x_Cygwin)
AC_DEFINE_UNQUOTED(SRCDIR,"$SRC_ROOT")
AC_DEFINE_UNQUOTED(BUILDDIR,"$BUILDDIR")

if test "z$EUID" = "z0" -a "`uname -o 2>/dev/null`" = "Cygwin"; then
    AC_MSG_ERROR([You must build LibreOffice as a normal user - not using an administrative account])
fi

# need sed in os checks...
AC_PATH_PROGS(SED, sed)
if test -z "$SED"; then
    AC_MSG_ERROR([install sed to run this script])
fi

# Set the ENABLE_LTO variable
# ===================================================================
AC_MSG_CHECKING([whether to use link-time optimization])
if test -n "$enable_lto" -a "$enable_lto" != "no"; then
    ENABLE_LTO="TRUE"
    AC_MSG_RESULT([yes])
    AC_DEFINE(STATIC_LINKING)
else
    ENABLE_LTO=""
    AC_MSG_RESULT([no])
fi
AC_SUBST(ENABLE_LTO)

dnl ===================================================================
dnl When building for Android, --with-android-ndk,
dnl --with-android-ndk-toolchain-version and --with-android-sdk are
dnl mandatory
dnl ===================================================================

AC_ARG_WITH(android-ndk,
    AS_HELP_STRING([--with-android-ndk],
        [Specify location of the Android Native Development Kit. Mandatory when building for Android.]),
,)

AC_ARG_WITH(android-ndk-toolchain-version,
    AS_HELP_STRING([--with-android-ndk-toolchain-version],
        [Specify which toolchain version to use, of those present in the
        Android NDK you are using. Mandatory if the NDK used has several
        toolchain versions for the host architecture you are building for.
        Possible values are 4.6, 4.8, clang3.3 and clang3.4. Only 4.8 has been
        tested for real...]), ,)

AC_ARG_WITH(android-sdk,
    AS_HELP_STRING([--with-android-sdk],
        [Specify location of the Android SDK. Mandatory when building for Android,
        or when building the Impress Remote Android app.]),
,)

ANDROID_NDK_HOME=
if test -n "$with_android_ndk"; then
    ANDROID_NDK_HOME=$with_android_ndk

    # Set up a lot of pre-canned defaults

    if test ! -f $ANDROID_NDK_HOME/RELEASE.TXT; then
        AC_MSG_ERROR([Unrecognized Android NDK. Missing RELEASE.TXT file in $ANDROID_NDK_HOME.])
    fi
    ANDROID_NDK_VERSION=`cut -f1 -d' ' <$ANDROID_NDK_HOME/RELEASE.TXT`

    case $ANDROID_NDK_VERSION in
    r9*|r10*)
        ;;
    *)
        AC_MSG_ERROR([Unsupported NDK version $ANDROID_NDK_VERSION, only r9* and r10* versions are supported])
        ;;
    esac

    if test $host_cpu = arm; then
        android_cpu=arm
        android_platform_prefix=$android_cpu-linux-androideabi
    elif test $host_cpu = aarch64; then
        android_cpu=aarch64
        android_platform_prefix=$android_cpu-linux-android
    elif test $host_cpu = mips; then
        android_cpu=mips
        android_platform_prefix=$android_cpu-linux-androideabi
    else
        # host_cpu is something like "i386" or "i686" I guess, NDK uses
        # "x86" in some contexts
        android_cpu=x86
        android_platform_prefix=$android_cpu
    fi

    case "$with_android_ndk_toolchain_version" in
    4.6|4.7|4.8|4.9)
        ANDROID_BINUTILS_DIR=$ANDROID_NDK_HOME/toolchains/$android_platform_prefix-$with_android_ndk_toolchain_version
        ANDROID_COMPILER_DIR=$ANDROID_BINUTILS_DIR
        ;;
    clang3.3|clang3.4)
        AC_MSG_WARN([Building with the Clang tool-chain is known to break in the bridges module, fix that please])
        ANDROID_BINUTILS_DIR=$ANDROID_NDK_HOME/toolchains/$android_platform_prefix-4.8
        ANDROID_COMPILER_DIR=$ANDROID_NDK_HOME/toolchains/llvm-${with_android_ndk_toolchain_version#clang}
        ANDROID_USING_CLANG=true
        ;;
    *)
        AC_MSG_ERROR([Unrecognized value for the --with-android-ndk-toolchain-version option])
    esac

    if test ! -d $ANDROID_BINUTILS_DIR; then
        AC_MSG_ERROR([No directory $ANDROID_BINUTILS_DIR])
    elif test $ANDROID_COMPILER_DIR != $ANDROID_BINUTILS_DIR -a ! -d $ANDROID_COMPILER_DIR; then
        AC_MSG_ERROR([No directory $ANDROID_COMPILER_DIR])
    fi

    # Check if there is a 64-bit tool-chain. Google provides a NDK with 64-bit tool-chain binaries in
    # NDK r8e and later, and for earlier NDKs it was possible to build one yourself. Using a 64-bit
    # linker is required if you compile large parts of the code with -g. A 32-bit linker just won't
    # manage to link the (app-specific) single huge .so that is built for the apps in
    # android/experimental/* if there is debug information in a significant part of the object files.
    # (A 64-bit ld.gold grows to much over 10 gigabytes of virtual space when linking such a .so if
    # all objects have been built with debug information.)
    toolchain_system='*'
    if test $build_os = linux-gnu; then
        ANDROID_COMPILER_BIN=$ANDROID_COMPILER_DIR/prebuilt/linux-x86/bin
        ANDROID_BINUTILS_PREBUILT_ROOT=$ANDROID_BINUTILS_DIR/prebuilt/linux-x86
        if test $build_cpu = x86_64; then
            if test -d $ANDROID_COMPILER_DIR/prebuilt/linux-x86_64; then
                ANDROID_COMPILER_BIN=$ANDROID_COMPILER_DIR/prebuilt/linux-x86_64/bin
            fi
            if test -d $ANDROID_BINUTILS_DIR/prebuilt/linux-x86_64; then
                ANDROID_BINUTILS_PREBUILT_ROOT=$ANDROID_BINUTILS_DIR/prebuilt/linux-x86_64
            fi
        fi
        ANDROID_BINUTILS_BIN=$ANDROID_BINUTILS_PREBUILT_ROOT/bin
    fi

    # This stays empty if there is just one version of the toolchain in the NDK
    ANDROID_NDK_TOOLCHAIN_VERSION_SUBDIR=
    case "`echo $ANDROID_NDK_HOME/toolchains/$android_cpu*-*/prebuilt/*/bin`" in
    */bin\ */bin*)
        # Trailing slash intentional and necessary, compare to how this is used
        if test -n "$ANDROID_USING_CLANG"; then
            ANDROID_NDK_TOOLCHAIN_VERSION_SUBDIR=4.8/
        else
            ANDROID_NDK_TOOLCHAIN_VERSION_SUBDIR="${with_android_ndk_toolchain_version}/"
        fi
        ;;
    esac

    ANDROID_API_LEVEL=15
    if test $host_cpu = arm; then
        android_gnu_prefix=arm-linux-androideabi
    elif test $host_cpu = aarch64; then
        android_gnu_prefix=aarch64-linux-android
        ANDROID_NDK_TOOLCHAIN_VERSION_SUBDIR="${with_android_ndk_toolchain_version}/"
        ANDROID_API_LEVEL=L
    elif test $host_cpu = mips; then
        android_gnu_prefix=mipsel-linux-android
    elif test $ANDROID_NDK_VERSION = r8; then
        # The prefix used for the x86 tool-chain changed between NDK r8 and r8b
        android_gnu_prefix=i686-android-linux
    else
        android_gnu_prefix=i686-linux-android
    fi

    ANDROID_ARCH=$android_cpu
    if test $host_cpu = arm; then
        ANDROID_APP_ABI=armeabi-v7a
        if test -n "$ANDROID_USING_CLANG"; then
            ANDROIDCFLAGS="-gcc-toolchain $ANDROID_BINUTILS_PREBUILT_ROOT"
            ANDROIDCFLAGS="$ANDROIDCFLAGS -target armv7-none-linux-androideabi"
            ANDROIDCFLAGS="$ANDROIDCFLAGS -no-canonical-prefixes"
        else
            :
        fi
        ANDROIDCFLAGS="$ANDROIDCFLAGS -mthumb"
        ANDROIDCFLAGS="$ANDROIDCFLAGS -march=armv7-a -mfloat-abi=softfp -mfpu=neon"
        ANDROIDCFLAGS="$ANDROIDCFLAGS -Wl,--fix-cortex-a8"
    elif test $host_cpu = aarch64; then
        ANDROID_APP_ABI=arm64-v8a
        ANDROID_ARCH=arm64
    elif test $host_cpu = mips; then
        ANDROID_APP_ABI=mips
        ANDROIDCFLAGS=""
    else # x86
        ANDROID_APP_ABI=x86
        ANDROIDCFLAGS="-march=atom"
    fi
    ANDROIDCFLAGS="$ANDROIDCFLAGS -ffunction-sections -fdata-sections"
    ANDROIDCFLAGS="$ANDROIDCFLAGS -L$ANDROID_NDK_HOME/sources/cxx-stl/gnu-libstdc++/${ANDROID_NDK_TOOLCHAIN_VERSION_SUBDIR}libs/$ANDROID_APP_ABI"
    ANDROIDCFLAGS="$ANDROIDCFLAGS --sysroot $ANDROID_NDK_HOME/platforms/android-${ANDROID_API_LEVEL}/arch-${ANDROID_ARCH}"

    if test -n "$ANDROID_USING_CLANG"; then
        ANDROIDCFLAGS="$ANDROIDCFLAGS -Qunused-arguments"
    else
        ANDROIDCFLAGS="$ANDROIDCFLAGS -Wno-psabi"
    fi

    test -z "$SYSBASE" && export SYSBASE=$ANDROID_NDK_HOME/platforms/android-${ANDROID_API_LEVEL}/arch-${ANDROID_ARCH}
    test -z "$AR" && AR=$ANDROID_BINUTILS_BIN/$android_gnu_prefix-ar
    test -z "$NM" && NM=$ANDROID_BINUTILS_BIN/$android_gnu_prefix-nm
    test -z "$OBJDUMP" && OBJDUMP=$ANDROID_BINUTILS_BIN/$android_gnu_prefix-objdump
    test -z "$RANLIB" && RANLIB=$ANDROID_BINUTILS_BIN/$android_gnu_prefix-ranlib
    test -z "$STRIP" && STRIP=$ANDROID_BINUTILS_BIN/$android_gnu_prefix-strip

    # When using the 4.6 or newer toolchain, use the gold linker
    case "$with_android_ndk_toolchain_version" in
    4.[[6789]]*|[[56789]].*|clang*)
        if test "$host_cpu" = arm -a "$ENABLE_LTO" != TRUE; then
            ANDROIDCFLAGS="$ANDROIDCFLAGS -fuse-ld=gold"
        fi
        ;;
    esac

    if test "$ENABLE_LTO" = TRUE; then
        # -flto comes from com_GCC_degs.mk, too, but we need to make sure it gets passed as part of
        # $CC and $CXX when building external libraries
        ANDROIDCFLAGS="$ANDROIDCFLAGS -flto"
    fi

    # gdbserver can be in different locations
    if test -f $ANDROID_NDK_HOME/toolchains/arm-linux-androideabi-4.4.3/prebuilt/gdbserver; then
        ANDROID_NDK_GDBSERVER=$ANDROID_NDK_HOME/toolchains/arm-linux-androideabi-4.4.3/prebuilt/gdbserver
    elif test -f $ANDROID_NDK_HOME/prebuilt/android-$android_cpu/gdbserver/gdbserver; then
        ANDROID_NDK_GDBSERVER=$ANDROID_NDK_HOME/prebuilt/android-$android_cpu/gdbserver/gdbserver
    elif test $android_cpu = aarch64; then
        ANDROID_NDK_GDBSERVER=$ANDROID_NDK_HOME/prebuilt/android-arm64/gdbserver/gdbserver
    else
        AC_MSG_ERROR([Can't find gdbserver for your Android target])
    fi

    if test $host_cpu = arm; then
        ANDROIDCXXFLAGS="$ANDROIDCFLAGS -I $ANDROID_NDK_HOME/sources/cxx-stl/gnu-libstdc++/${ANDROID_NDK_TOOLCHAIN_VERSION_SUBDIR}include -I$ANDROID_NDK_HOME/sources/cxx-stl/gnu-libstdc++/${ANDROID_NDK_TOOLCHAIN_VERSION_SUBDIR}libs/armeabi-v7a/include"
    elif test $host_cpu = mips; then
        ANDROIDCXXFLAGS="$ANDROIDCFLAGS -I $ANDROID_NDK_HOME/sources/cxx-stl/gnu-libstdc++/${ANDROID_NDK_TOOLCHAIN_VERSION_SUBDIR}include -I$ANDROID_NDK_HOME/sources/cxx-stl/gnu-libstdc++/${ANDROID_NDK_TOOLCHAIN_VERSION_SUBDIR}libs/mips/include"
    else # x86
        ANDROIDCXXFLAGS="$ANDROIDCFLAGS -I $ANDROID_NDK_HOME/sources/cxx-stl/gnu-libstdc++/${ANDROID_NDK_TOOLCHAIN_VERSION_SUBDIR}include -I$ANDROID_NDK_HOME/sources/cxx-stl/gnu-libstdc++/${ANDROID_NDK_TOOLCHAIN_VERSION_SUBDIR}libs/x86/include"
    fi

    if test -z "$CC"; then
        case "$with_android_ndk_toolchain_version" in
        4.*)
            CC="$ANDROID_COMPILER_BIN/$android_gnu_prefix-gcc $ANDROIDCFLAGS"
            ;;
        clang*)
            CC="$ANDROID_COMPILER_BIN/clang $ANDROIDCFLAGS"
        esac
    fi
    if test -z "$CXX"; then
        case "$with_android_ndk_toolchain_version" in
        4.*)
            CXX="$ANDROID_COMPILER_BIN/$android_gnu_prefix-g++ $ANDROIDCXXFLAGS"
            ;;
        clang*)
            CXX="$ANDROID_COMPILER_BIN/clang++ $ANDROIDCXXFLAGS"
            ;;
        esac
    fi
fi
AC_SUBST(ANDROID_NDK_GDBSERVER)
AC_SUBST(ANDROID_APP_ABI)

dnl ===================================================================
dnl Also --with-android-sdk is mandatory
dnl ===================================================================
ANDROID_SDK_HOME=
if test -n "$with_android_sdk"; then
    ANDROID_SDK_HOME=$with_android_sdk
    PATH="$ANDROID_SDK_HOME/platform-tools:$ANDROID_SDK_HOME/tools:$PATH"
fi
AC_SUBST(ANDROID_SDK_HOME)

dnl ===================================================================
dnl The following is a list of supported systems.
dnl Sequential to keep the logic very simple
dnl These values may be checked and reset later.
dnl ===================================================================
#defaults unless the os test overrides this:
test_randr=yes
test_xrender=yes
test_cups=yes
test_dbus=yes
test_fontconfig=yes

# Default values, as such probably valid just for Linux, set
# differently below just for Mac OSX,but at least better than
# hardcoding these as we used to do. Much of this is duplicated also
# in solenv for old build system and for gbuild, ideally we should
# perhaps define stuff like this only here in configure.ac?

LINKFLAGSSHL="-shared"
PICSWITCH="-fpic"
DLLPOST=".so"

LINKFLAGSNOUNDEFS="-Wl,-z,defs"

INSTROOTSUFFIX=
SDKDIRNAME=sdk

case "$host_os" in

solaris*)
    test_gtk=yes
    build_gstreamer=yes
    build_gstreamer_0_10=yes
    test_tde=yes
    test_kde=yes
    test_freetype=yes
    test_gstreamer=yes
    _os=SunOS

    dnl ===========================================================
    dnl Check whether we're using Solaris 10 - SPARC or Intel.
    dnl ===========================================================
    AC_MSG_CHECKING([the Solaris operating system release])
    _os_release=`echo $host_os | $SED -e s/solaris2\.//`
    if test "$_os_release" -lt "10"; then
        AC_MSG_ERROR([use Solaris >= 10 to build LibreOffice])
    else
        AC_MSG_RESULT([ok ($_os_release)])
    fi

    dnl Check whether we're using a SPARC or i386 processor
    AC_MSG_CHECKING([the processor type])
    if test "$host_cpu" = "sparc" -o "$host_cpu" = "i386"; then
        AC_MSG_RESULT([ok ($host_cpu)])
    else
        AC_MSG_ERROR([only SPARC and i386 processors are supported])
    fi
    ;;

linux-gnu*|k*bsd*-gnu*)
    test_gtk=yes
    build_gstreamer=yes
    build_gstreamer_0_10=yes
    test_tde=yes
    test_kde=yes
    test_kde4=yes
    test_freetype=yes
    _os=Linux
    ;;

gnu)
    test_randr=no
    test_xrender=no
    _os=GNU
     ;;

cygwin*|interix*|mingw32*)

    # When building on Windows normally with MSVC under Cygwin,
    # configure thinks that the host platform (the platform the
    # built code will run on) is Cygwin, even if it obviously is
    # Windows, which in Autoconf terminology is called
    # "mingw32". (Which is misleading as MinGW is the name of the
    # tool-chain, not an operating system.)

    # Somewhat confusing, yes. But this configure script doesn't
    # look at $host etc that much, it mostly uses its own $_os
    # variable, set here in this case statement.

    # When cross-compiling to Windows from Unix, the host platform
    # is "mingw32" (because in that case it is the MinGW
    # tool-chain that is used).

    test_cups=no
    test_dbus=no
    test_randr=no
    test_xrender=no
    test_freetype=no
    test_fontconfig=no
    _os=WINNT

    DLLPOST=".dll"
    LINKFLAGSNOUNDEFS=

    # If the host OS matches "mingw32*", that means we are using the
    # MinGW cross-compiler, because we don't see the point in building
    # LibreOffice using MinGW on Windows. If you want to build on
    # Windows, use MSVC. If you want to use MinGW, surely you want to
    # cross-compile (from Linux or some other Unix).

    case "$host_os" in
    mingw32*)
        WITH_MINGW=yes
        if test -z "$CC"; then
            CC="$host_cpu-$host_vendor-$host_os-gcc"
        fi
        if test -z "$CXX"; then
            CXX="$host_cpu-$host_vendor-$host_os-g++"
        fi
        ;;
    esac
    ;;

darwin*) # Mac OS X or iOS
    test_gtk=yes
    test_randr=no
    test_xrender=no
    test_freetype=no
    test_fontconfig=no
    test_dbus=no
    if test "$host_cpu" = "arm"; then
        _os=iOS
        test_gtk=no
        test_cups=no
    else
        _os=Darwin
        INSTROOTSUFFIX=/$PRODUCTNAME.app/Contents
        SDKDIRNAME=AC_PACKAGE_NAME${PRODUCTVERSION}_SDK
    fi
    enable_systray=no
    # See comment above the case "$host_os"
    LINKFLAGSSHL="-dynamiclib -single_module"

    # -fPIC is default
    PICSWITCH=""

    DLLPOST=".dylib"

    # -undefined error is the default
    LINKFLAGSNOUNDEFS=""
;;

freebsd*)
    test_gtk=yes
    build_gstreamer=yes
    build_gstreamer_0_10=yes
    test_tde=yes
    test_kde=yes
    test_kde4=yes
    test_freetype=yes
    AC_MSG_CHECKING([the FreeBSD operating system release])
    if test -n "$with_os_version"; then
        OSVERSION="$with_os_version"
    else
        OSVERSION=`/sbin/sysctl -n kern.osreldate`
    fi
    AC_MSG_RESULT([found OSVERSION=$OSVERSION])
    AC_MSG_CHECKING([which thread library to use])
    if test "$OSVERSION" -lt "500016"; then
        PTHREAD_CFLAGS="-D_THREAD_SAFE"
        PTHREAD_LIBS="-pthread"
    elif test "$OSVERSION" -lt "502102"; then
        PTHREAD_CFLAGS="-D_THREAD_SAFE"
        PTHREAD_LIBS="-lc_r"
    else
        PTHREAD_CFLAGS=""
        PTHREAD_LIBS="-pthread"
    fi
    AC_MSG_RESULT([$PTHREAD_LIBS])
    _os=FreeBSD
    ;;

*netbsd*)
    test_gtk=yes
    build_gstreamer=yes
    build_gstreamer_0_10=yes
    test_tde=no
    test_kde=no
    test_kde4=yes
    test_freetype=yes
    PTHREAD_LIBS="-pthread -lpthread"
    _os=NetBSD
    ;;

aix*)
    test_randr=no
    test_freetype=yes
    PTHREAD_LIBS=-pthread
    _os=AIX
    ;;

openbsd*)
    test_gtk=yes
    test_tde=yes
    test_kde=yes
    test_freetype=yes
    PTHREAD_CFLAGS="-D_THREAD_SAFE"
    PTHREAD_LIBS="-pthread"
    _os=OpenBSD
    ;;

dragonfly*)
    test_gtk=yes
    build_gstreamer=yes
    build_gstreamer_0_10=yes
    test_tde=yes
    test_kde=yes
    test_kde4=yes
    test_freetype=yes
    PTHREAD_LIBS="-pthread"
    _os=DragonFly
    ;;

linux-android*)
    build_gstreamer=no
    build_gstreamer_0_10=no
    enable_lotuswordpro=no
    enable_mpl_subset=yes
    enable_opengl=no
    enable_coinmp=yes
    enable_report_builder=no
    with_theme="tango"
    test_cups=no
    test_dbus=no
    test_fontconfig=no
    test_freetype=no
    test_gtk=no
    test_tde=no
    test_kde=no
    test_kde4=no
    test_randr=no
    test_xrender=no
    _os=Android

    if test -z "$with_android_ndk"; then
        AC_MSG_ERROR([the --with-android-ndk option is mandatory])
    fi

    if test -z "$with_android_ndk_toolchain_version"; then
        AC_MSG_ERROR([the --with-android-ndk-toolchain-version option is mandatory])
    fi

    # Verify that the NDK and SDK options are proper
    if test ! -f "$ANDROID_NDK_HOME/platforms/android-${ANDROID_API_LEVEL}/arch-${ANDROID_ARCH}/usr/lib/libc.a"; then
        AC_MSG_ERROR([the --with-android-ndk option does not point to an Android NDK])
    fi

    AC_DEFINE(HAVE_FT_FACE_GETCHARVARIANTINDEX)
    BUILD_TYPE="$BUILD_TYPE FONTCONFIG FREETYPE"
    ;;

*)
    AC_MSG_ERROR([$host_os operating system is not suitable to build LibreOffice for!])
    ;;
esac

if echo "$host_os" | grep -q linux-android ; then
    if test -z "$with_android_sdk"; then
        AC_MSG_ERROR([the --with-android-sdk option is mandatory])
    fi

    if test ! -d "$ANDROID_SDK_HOME/platforms"; then
        AC_MSG_ERROR([the --with-android-sdk option does not point to an Android SDK])
    fi
fi

if test "$_os" = "AIX"; then
    AC_PATH_PROG(GAWK, gawk)
    if test -z "$GAWK"; then
        AC_MSG_ERROR([gawk not found in \$PATH])
    fi
fi

AC_SUBST(SDKDIRNAME)

AC_SUBST(WITH_MINGW)
AC_SUBST(PTHREAD_CFLAGS)
AC_SUBST(PTHREAD_LIBS)

###############################################################################
# Extensions switches --enable/--disable
###############################################################################
# By default these should be enabled unless having extra dependencies.
# If there is extra dependency over configure options then the enable should
# be automagic based on whether the requiring feature is enabled or not.
# All this options change anything only with --enable-extension-integration.

# The name of this option and its help string makes it sound as if
# extensions are built anyway, just not integrated in the installer,
# if you use --disable-extension-integration. Is that really the
# case?

AC_ARG_ENABLE(extension-integration,
    AS_HELP_STRING([--disable-extension-integration],
        [Disable integration of the built extensions in the installer of the
         product. Use this switch to disable the integration.])
)

AC_ARG_ENABLE(export,
    AS_HELP_STRING([--disable-export],
        [Disable (some) code for document export. Useful when building viewer-only apps that lack
         save/export functionality, to avoid having an excessive amount of code and data used
         only for exporrt linked in. Work in progress, use only if you are hacking on it.])
)

AC_ARG_ENABLE(avmedia,
    AS_HELP_STRING([--disable-avmedia],
        [Disable displaying and inserting AV media in documents. Work in progress, use only if you are hacking on it.])
)

AC_ARG_ENABLE(database-connectivity,
    AS_HELP_STRING([--disable-database-connectivity],
        [Disable various database connectivity. Work in progress, use only if you are hacking on it.])
)

# This doesn't mean not building (or "integrating") extensions
# (although it probably should; i.e. it should imply
# --disable-extension-integration I guess), it means not supporting
# any extension mechanism at all
AC_ARG_ENABLE(extensions,
    AS_HELP_STRING([--disable-extensions],
        [Disable all add-on extension functionality. Work in progress, use only if you are hacking on it.])
)

AC_ARG_ENABLE(scripting,
    AS_HELP_STRING([--disable-scripting],
        [Disable BASIC, Java and Python. Work in progress, use only if you are hacking on it.])
)

AC_ARG_ENABLE(epm,
    AS_HELP_STRING([--enable-epm],
        [LibreOffice includes self-packaging code, that requires epm, however epm is
         useless for large scale package building.])
)

AC_ARG_ENABLE(odk,
    AS_HELP_STRING([--disable-odk],
        [LibreOffice includes an ODK, office development kit which some packagers may
         wish to build without.])
)

AC_ARG_ENABLE(evolution2,
    AS_HELP_STRING([--enable-evolution2],
        [Allows the built-in evolution 2 addressbook connectivity build to be
         enabled.])
)

AC_ARG_ENABLE(directx,
    AS_HELP_STRING([--disable-directx],
        [Remove DirectX implementation for the new XCanvas interface.
         The DirectX support requires more stuff installed on Windows to
         compile. (DirectX SDK, GDI+ libs)])
)

AC_ARG_ENABLE(activex,
    AS_HELP_STRING([--disable-activex],
        [Disable the use of ActiveX for a Windows build.
        This switch is mandatory when using an Express edition of Visual Studio.])
)

AC_ARG_ENABLE(avahi,
    AS_HELP_STRING([--enable-avahi],
        [Determines whether to use Avahi to advertise Impress to remote controls.]),
,)

AC_ARG_ENABLE(werror,
    AS_HELP_STRING([--enable-werror],
        [Turn warnings to errors. (Has no effect in modules where the treating
         of warnings as errors is disabled explicitly.)]),
,)

AC_ARG_ENABLE(assert-always-abort,
    AS_HELP_STRING([--enable-assert-always-abort],
        [make assert() abort even in release code.]),
,)

AC_ARG_ENABLE(dbgutil,
    AS_HELP_STRING([--enable-dbgutil],
        [Provide debugging support from --enable-debug and include additional debugging
         utilities such as object counting or more expensive checks.
         This is the recommended option for developers.
         Note that this makes the build ABI incompatible, it is not possible to mix object
         files or libraries from a --enable-dbgutil and a --disable-dbgutil build.]))

AC_ARG_ENABLE(debug,
    AS_HELP_STRING([--enable-debug],
        [Include debugging information, disable compiler optimization and inlining plus
         extra debugging code like assertions. Extra large build! (enables -g compiler flag).]))

AC_ARG_ENABLE(sal-log,
    AS_HELP_STRING([--enable-sal-log],
        [Make SAL_INFO and SAL_WARN calls do something even in a non-debug build.]))

AC_ARG_ENABLE(selective-debuginfo,
    AS_HELP_STRING([--enable-selective-debuginfo],
        [If --enable-debug or --enable-dbgutil is used, build debugging information
         (-g compiler flag) only for the specified gbuild build targets
         (where all means everything, - prepended means not to enable, / appended means
         everything in the directory; there is no ordering, more specific overrides
         more general, and disabling takes precedence).
         Example: --enable-selective-debuginfo="all -sw/ -Library_sc".]))

AC_ARG_ENABLE(symbols,
    AS_HELP_STRING([--enable-symbols],
        [Include debugging symbols in output while preserve optimization.
         This enables -g compiler flag for GCC or equivalent,
         without changing anything else compared to productive code.]))

AC_ARG_ENABLE(runtime-optimizations,
    AS_HELP_STRING([--disable-runtime-optimizations],
        [Statically disable certain runtime optimizations (like rtl/alloc.h or
         JVM JIT) that are known to interact badly with certain dynamic analysis
         tools (like -fsanitize=address or Valgrind).  By default, disabled iff
         CC contains "-fsanitize=*".  (For Valgrind, those runtime optimizations
         are typcially disabled dynamically via RUNNING_ON_VALGRIND.)]))

AC_ARG_ENABLE(compiler-plugins,
    AS_HELP_STRING([--enable-compiler-plugins],
        [Enable compiler plugins that will perform additional checks during
         building. Enabled automatically by --enable-dbgutil.]))

AC_ARG_ENABLE(ooenv,
    AS_HELP_STRING([--disable-ooenv],
        [Disable ooenv for the instdir installation.]))

AC_ARG_ENABLE(lto,
    AS_HELP_STRING([--enable-lto],
        [Enable link-time optimization. Suitable for (optimised) product builds. Building might take
         longer but libraries and executables are optimized for speed. (For GCC, best to use GCC 4.6
         or later and 'gold' linker. Works fine (and is turned on automatically in an optimising
         build) for MSVC, otherwise experimental work in progress that shouldn't be used unless you
         are working on it.)]))

AC_ARG_ENABLE(crashdump,
    AS_HELP_STRING([--enable-crashdump],
        [Enable the crashdump feature.]))

AC_ARG_ENABLE(python,
    AS_HELP_STRING([--enable-python=<no/auto/system/internal/fully-internal>],
        [Enables or disables Python support at run-time and build-time.
         Also specifies what Python to use. 'auto' is the default.
         'fully-internal' even forces the internal version for uses of Python
         during the build.]))

AC_ARG_ENABLE(gtk,
    AS_HELP_STRING([--disable-gtk],
        [Determines whether to use Gtk+ vclplug on platforms where Gtk+ is available.]),
,enable_gtk=yes)

AC_ARG_ENABLE(gtk3,
    AS_HELP_STRING([--enable-gtk3],
        [Determines whether to use Gtk+ 3.0 vclplug on platforms where Gtk+ 3.0 is available.]),
,enable_gtk3=no)

AC_ARG_ENABLE(systray,
    AS_HELP_STRING([--disable-systray],
        [Determines whether to build the systray quickstarter.]),
,enable_systray=yes)

AC_ARG_ENABLE(split-app-modules,
    AS_HELP_STRING([--enable-split-app-modules],
        [Split file lists for app modules, e.g. base, calc.
         Has effect only with make distro-pack-install]),
,)

AC_ARG_ENABLE(split-opt-features,
    AS_HELP_STRING([--enable-split-opt-features],
        [Split file lists for some optional features, .e.g. pyuno, testtool.
         Has effect only with make distro-pack-install]),
,)

AC_ARG_ENABLE(opengl,
    AS_HELP_STRING([--disable-opengl],
        [Determines whether to build the OpenGL 3D slide transitions and canvas component.]))

AC_ARG_ENABLE(opencl,
    AS_HELP_STRING([--disable-opencl],
        [Determines whether to build with OpenCL support.]))

AC_ARG_ENABLE(dbus,
    AS_HELP_STRING([--disable-dbus],
        [Determines whether to enable features that depend on dbus.
         e.g. Presentation mode screensaver control, bluetooth presentation control]),
,enable_dbus=yes)

AC_ARG_ENABLE(packagekit,
    AS_HELP_STRING([--enable-packagekit],
        [Determines whether to enable features using packagekit.
         Right now that is auto font install]),
,)

AC_ARG_ENABLE(sdremote,
    AS_HELP_STRING([--disable-sdremote],
        [Determines whether to enable Impress remote control (i.e. the server component).]),
,enable_sdremote=yes)

AC_ARG_ENABLE(sdremote-bluetooth,
    AS_HELP_STRING([--disable-sdremote-bluetooth],
        [Determines whether to build sdremote with bluetooth support.
         Requires dbus on Linux.]))

AC_ARG_ENABLE(gconf,
    AS_HELP_STRING([--disable-gconf],
        [Determines whether to use the GConf support.]),
,enable_gconf=yes)

AC_ARG_ENABLE(gnome-vfs,
    AS_HELP_STRING([--enable-gnome-vfs],
        [Determines whether to use the Gnome Virtual Filing System on platforms
         where that VFS is available.]),
,enable_gnome_vfs=no)

AC_ARG_ENABLE(gio,
    AS_HELP_STRING([--disable-gio],
        [Determines whether to use the GIO support.]),
,enable_gio=yes)

AC_ARG_ENABLE(telepathy,
    AS_HELP_STRING([--enable-telepathy],
        [Determines whether to enable Telepathy for collaboration.]),
,enable_telepathy=no)

AC_ARG_ENABLE(tde,
    AS_HELP_STRING([--enable-tde],
        [Determines whether to use TQt/TDE vclplug on platforms where TQt and
         TDE are available.]),
,)

AC_ARG_ENABLE(tdeab,
    AS_HELP_STRING([--disable-tdeab],
        [Disable the TDE address book support.]),
,
    if test "$enable_tde" = "yes"; then
        enable_tdeab=yes
    fi
)

AC_ARG_ENABLE(kde,
    AS_HELP_STRING([--enable-kde],
        [Determines whether to use Qt3/KDE3 vclplug on platforms where Qt3 and
         KDE3 are available.]),
,)

AC_ARG_ENABLE(kdeab,
    AS_HELP_STRING([--disable-kdeab],
        [Disable the KDE3 address book support.]),
,
    if test "$enable_kde" = "yes"; then
        enable_kdeab=yes
    fi
)

AC_ARG_ENABLE(kde4,
    AS_HELP_STRING([--enable-kde4],
        [Determines whether to use Qt4/KDE4 vclplug on platforms where Qt4 and
         KDE4 are available. May be used with --enable-kde if you want to support
         both KDE3 and KDE4.]),
,)

AC_ARG_ENABLE(headless,
    AS_HELP_STRING([--enable-headless],
        [Disable building of GUIs to reduce dependencies. Useful for
        server usage. Work in progress, use only if you are hacking on
        it. Not related to the --headless option.]), ,)

AC_ARG_ENABLE(randr,
    AS_HELP_STRING([--disable-randr],
        [Disable RandR support in the vcl project.]),
,enable_randr=yes)

AC_ARG_ENABLE(randr-link,
    AS_HELP_STRING([--disable-randr-link],
        [Disable linking with libXrandr, instead dynamically open it at runtime.]),
,enable_randr_link=yes)

AC_ARG_ENABLE(gstreamer,
    AS_HELP_STRING([--disable-gstreamer],
        [Disable building with the new gstreamer 1.0 avmedia backend.]),
,enable_gstreamer=yes)

AC_ARG_ENABLE(gstreamer-0-10,
    AS_HELP_STRING([--enable-gstreamer-0-10],
        [Enable building the gstreamer 0.10 avmedia backend.]),
,enable_gstreamer_0_10=no)

AC_ARG_ENABLE(vlc,
    AS_HELP_STRING([--enable-vlc],
        [Enable building with the VLC avmedia backend.]),
,enable_vlc=no)

AC_ARG_ENABLE([eot],
    [AS_HELP_STRING([--enable-eot],
        [Enable support for Embedded OpenType fonts.])],
, [enable_eot=no])

AC_ARG_ENABLE(cve-tests,
    AS_HELP_STRING([--disable-cve-tests],
        [Prevent CVE tests to be executed]),
,)

AC_ARG_ENABLE(chart-tests,
    AS_HELP_STRING([--enable-chart-tests],
        [Executes chart XShape tests. In a perfect world these tests would be
         stable and everyone could run them, in reality it is best to run them
         only on a few machines that are known to work and maintained by people
         who can judge if a test failure is a regression or not.]),
,)

AC_ARG_ENABLE(build-unowinreg,
    AS_HELP_STRING([--enable-build-unowinreg],
        [Do not use the prebuilt unowinreg.dll. Build it instead. The MinGW C++
         compiler is needed on Linux.])
    [
                          Usage:     --enable-build-unowinreg
    ],
,)

AC_ARG_ENABLE(verbose,
    AS_HELP_STRING([--enable-verbose],
        [Increase build verbosity.])[
  --disable-verbose       Decrease build verbosity.],
,)

AC_ARG_ENABLE(dependency-tracking,
    AS_HELP_STRING([--enable-dependency-tracking],
        [Do not reject slow dependency extractors.])[
  --disable-dependency-tracking
                          Disables generation of dependency information.
                          Speed up one-time builds.],
,)

AC_ARG_ENABLE(icecream,
    AS_HELP_STRING([--enable-icecream],
        [Use the 'icecream' distributed compiling tool to speedup the compilation.
         It defaults to /opt/icecream for the location of the icecream gcc/g++
         wrappers, you can override that using --with-gcc-home=/the/path switch.]),
,)

AC_ARG_ENABLE(cups,
    AS_HELP_STRING([--disable-cups],
        [Do not build cups support.])
)

AC_ARG_ENABLE(ccache,
    AS_HELP_STRING([--disable-ccache],
        [Do not try to use ccache automatically.
         By default, we will try to detect if ccache is available; in that case if
         CC/CXX are not yet set, and --enable-icecream is not given, we
         attempt to use ccache. --disable-ccache disables ccache completely.
]),
,)

AC_ARG_ENABLE(64-bit,
    AS_HELP_STRING([--enable-64-bit],
        [Build a 64-bit LibreOffice on platforms where the normal build is 32-bit.
         At the moment meaningful only for iOS and Windows. On Windows this option is
         experimental and possibly quite broken, and you should use it only if you are
         hacking on 64-bitness support.]), ,)

AC_ARG_ENABLE(extra-gallery,
    AS_HELP_STRING([--enable-extra-gallery],
        [Add extra gallery content.]),
,)

AC_ARG_ENABLE(extra-template,
    AS_HELP_STRING([--enable-extra-template],
        [Add extra template content.]),
,)

AC_ARG_ENABLE(extra-sample,
    AS_HELP_STRING([--enable-extra-sample],
        [Add extra sample content.]),
,)

AC_ARG_ENABLE(extra-font,
    AS_HELP_STRING([--enable-extra-font],
        [Add extra font content.]),
,)

AC_ARG_ENABLE(online-update,
    AS_HELP_STRING([--enable-online-update],
        [Enable the online update service that will check for new versions of
         LibreOffice. By default, it is on on Windows and Mac, and off on Linux.]),
,)

AC_ARG_ENABLE(release-build,
    AS_HELP_STRING([--enable-release-build],
        [Enable release build.
         See http://wiki.documentfoundation.org/DevBuild]),
,)

AC_ARG_ENABLE(windows-build-signing,
    AS_HELP_STRING([--enable-windows-build-signing],
        [Enable signing of windows binaries (*.exe, *.dll)]),
,)

AC_ARG_ENABLE(silent-msi,
    AS_HELP_STRING([--enable-silent-msi],
        [Enable MSI with LIMITUI=1 (silent install).]),
,)

AC_ARG_ENABLE(macosx-retina,
    AS_HELP_STRING([--enable-macosx-retina],
        [deprecated... enabled by default now. Kept for backward compat only.
         ]),
,)

AC_ARG_ENABLE(macosx-code-signing,
    AS_HELP_STRING([--enable-macosx-code-signing=<identity>],
        [Sign executables, dylibs, frameworks and the app bundle. If you
         don't provide an identity the first suitable certificate
         in your keychain is used.]),
,)

AC_ARG_ENABLE(macosx-package-signing,
    AS_HELP_STRING([--enable-macosx-package-signing=<identity>],
        [Create a .pkg suitable for uploading to the Mac App Store and sign
         it. If you don't provide an identity the first suitable certificate
         in your keychain is used.]),
,)

AC_ARG_ENABLE(macosx-sandbox,
    AS_HELP_STRING([--enable-macosx-sandbox],
        [Make the app bundle run in a sandbox. Requires code signing.
         Is required by apps distributed in the Mac App Store, and implies
         adherence to App Store rules.]),
,)

AC_ARG_WITH(macosx-bundle-identifier,
    AS_HELP_STRING([--with-macosx-bundle-identifier=tld.mumble.orifice.TheOffice],
        [Define the OS X bundle identifier. Default is the somewhat weird
         org.libreoffice.script ("script", huh?).]),
,with_macosx_bundle_identifier=org.libreoffice.script)

AC_ARG_WITH(macosx-app-name,
    AS_HELP_STRING([--with-macosx-app-name='My Own Office Suite'],
        [Define the OS X app name. Default is AC_PACKAGE_NAME.]),
,with_macosx_app_name=$PRODUCTNAME)

AC_ARG_ENABLE(ios-simulator,
    AS_HELP_STRING([--enable-ios-simulator],
        [Build for the iOS Simulator, not iOS device.]),
,)

AC_ARG_ENABLE(readonly-installset,
    AS_HELP_STRING([--enable-readonly-installset],
        [Prevents any attempts by LibreOffice to write into its installation. That means
         at least that no "system-wide" extensions can be added. Experimental work in
         progress.]),
,)

AC_ARG_ENABLE(postgresql-sdbc,
    AS_HELP_STRING([--disable-postgresql-sdbc],
        [Disable the build of the PostgreSQL-SDBC driver.])
)

AC_ARG_ENABLE(lotuswordpro,
    AS_HELP_STRING([--disable-lotuswordpro],
        [Disable the build of the Lotus Word Pro filter.]),
,enable_lotuswordpro=yes)

AC_ARG_ENABLE(firebird-sdbc,
    AS_HELP_STRING([--disable-firebird-sdbc],
        [Disable the build of the Firebird-SDBC driver if it doesn't compile for you.]),
,enable_firebird_sdbc=yes)

AC_ARG_ENABLE(winegcc,
    AS_HELP_STRING([--enable-winegcc],
        [Enable use of winegcc during the build, in order to create msi* tools
         needed for MinGW cross-compilation.]),
)

AC_ARG_ENABLE(liblangtag,
    AS_HELP_STRING([--disable-liblangtag],
        [Disable use of liblangtag, and instead use an own simple
         implementation.]),
)

AC_ARG_ENABLE(bogus-pkg-config,
    AS_HELP_STRING([--enable-bogus-pkg-config],
        [MACOSX only: on MacOSX pkg-config can cause trouble. by default if one is found in the PATH, an error is issued. This flag turn that error into a warning.]),
)

AC_ARG_ENABLE(openssl,
    AS_HELP_STRING([--disable-openssl],
        [Disable using libssl/libcrypto from OpenSSL. If disabled,
         components will either use GNUTLS or NSS. Work in progress,
         use only if you are hacking on it.]),
,enable_openssl=yes)

AC_ARG_ENABLE(library-bin-tar,
    AS_HELP_STRING([--enable-library-bin-tar],
        [Enable the building and reused of tarball of binary build for some 'external' libraries
        Some libraries can save their build result in a tarball
        stored in TARFILE_LOCATION. that binary tarball is
        uniquely identified by the source tarbal,
        the content of the config_host.mk file and the content
        of the top-level directory in core for that library
        If this option is enabled, then if such a tarfile exist, it will be untarred
        isntead of the source tarfile, and the build step will be skipped for that
        library.
        If a proper tarfile does not exist, then the normal source-based
        build is done for taht library and a proper binary tarfile is created
        for the next time.]),
)

AC_ARG_ENABLE(gltf,
    AS_HELP_STRING([--disable-gltf],
        [Determines whether to build libraries related to glTF 3D model rendering.]))

AC_ARG_ENABLE(collada,
    AS_HELP_STRING([--disable-collada],
        [Disable collada support (Rendering 3D models stored in *.dae and *.kmz format).]))

dnl ===================================================================
dnl Optional Packages (--with/without-)
dnl ===================================================================

AC_ARG_WITH(gnu-patch,
    AS_HELP_STRING([--with-gnu-patch],
        [Specify location of GNU patch on Solaris or FreeBSD.]),
,)

AC_ARG_WITH(build-platform-configure-options,
    AS_HELP_STRING([--with-build-platform-configure-options],
        [Specify options for the configure script run for the *build* platform in a cross-compilation]),
,)

AC_ARG_WITH(gnu-cp,
    AS_HELP_STRING([--with-gnu-cp],
        [Specify location of GNU cp on Solaris or FreeBSD.]),
,)

AC_ARG_WITH(external-tar,
    AS_HELP_STRING([--with-external-tar=<TARFILE_PATH>],
        [Specify an absolute path of where to find (and store) tarfiles.]),
    TARFILE_LOCATION=$withval ,
)

AC_ARG_WITH(referenced-git,
    AS_HELP_STRING([--with-referenced-git=<OTHER_CHECKOUT_DIR>],
        [Specify another checkout directory to reference. This makes use of
                 git submodule update --reference, and saves a lot of diskspace
                 when having multiple trees side-by-side.]),
    GIT_REFERENCE_SRC=$withval ,
)

AC_ARG_WITH(linked-git,
    AS_HELP_STRING([--with-linked-git=<submodules repo basedir>],
        [Specify a directory where the repositories of submodules are located.
         This uses a method similar to git-new-workdir to get submodules.]),
    GIT_LINK_SRC=$withval ,
)

AC_ARG_WITH(galleries,
    AS_HELP_STRING([--with-galleries],
        [Specify how galleries should be built. It is possible either to
         build these internally from source ("build"),
         or to disable them ("no")]),
)

AC_ARG_WITH(theme,
    AS_HELP_STRING([--with-theme="theme1 theme2..."],
        [Choose which themes to include. By default those themes with an '*' are included.
         Possible choices: *crystal, *galaxy, *hicontrast, human, industrial, *oxygen, *sifr, *tango, *tango_testing.]),
,)

AC_ARG_WITH(helppack-integration,
[
  --without-helppack-integration      It will not integrate the helppacks to the installer
                          of the product.
                          Please use this switch to use the online help or separate help packages.],
,)

AC_ARG_WITH(fonts,
    AS_HELP_STRING([--without-fonts],
        [LibreOffice includes some third-party fonts to provide a reliable basis for
         help content, templates, samples, etc. When these fonts are already
         known to be available on the system then you should use this option.]),
,)

AC_ARG_WITH(epm,
    AS_HELP_STRING([--with-epm],
        [Decides which epm to use. Default is to use the one from the system if
         one is built. When either this is not there or you say =internal epm
         will be built.]),
,)

AC_ARG_WITH(package-format,
    AS_HELP_STRING([--with-package-format],
        [Specify package format(s) for LibreOffice installation sets. The
         implicit --without-package-format leads to no installation sets being
         generated. Possible values: aix, archive, bsd, deb, dmg,
         installed, msi, pkg, and rpm.
         Example: --with-package-format='deb rpm']),
,)

AC_ARG_WITH(system-libs,
    AS_HELP_STRING([--with-system-libs],
        [Use libraries already on system -- enables all --with-system-* flags.]),
,)

AC_ARG_WITH(system-headers,
    AS_HELP_STRING([--with-system-headers],
        [Use headers already on system -- enables all --with-system-* flags for
         external packages whose headers are the only entities used i.e.
         boost/vigra/odbc/sane-header(s).]),,
    [with_system_headers="$with_system_libs"])

AC_ARG_WITH(system-jars,
    AS_HELP_STRING([--without-system-jars],
        [When building with --with-system-libs, also the needed jars are expected
         on the system. Use this to disable that]),,
    [with_system_jars="$with_system_libs"])

AC_ARG_WITH(myspell-dicts,
    AS_HELP_STRING([--with-myspell-dicts],
        [Adds myspell dictionaries to the LibreOffice installation set]),
,)

AC_ARG_WITH(system-dicts,
    AS_HELP_STRING([--without-system-dicts],
        [Do not use dictionaries from system paths.]),
,)

AC_ARG_WITH(external-dict-dir,
    AS_HELP_STRING([--with-external-dict-dir],
        [Specify external dictionary dir.]),
,)

AC_ARG_WITH(external-hyph-dir,
    AS_HELP_STRING([--with-external-hyph-dir],
        [Specify external hyphenation pattern dir.]),
,)

AC_ARG_WITH(external-thes-dir,
    AS_HELP_STRING([--with-external-thes-dir],
        [Specify external thesaurus dir.]),
,)

AC_ARG_WITH(system-zlib,
    AS_HELP_STRING([--with-system-zlib],
        [Use zlib already on system.]),,
    [with_system_zlib=auto])

AC_ARG_WITH(system-jpeg,
    AS_HELP_STRING([--with-system-jpeg],
        [Use jpeg already on system.]),,
    [with_system_jpeg=auto])

AC_ARG_WITH(system-libgltf,
    AS_HELP_STRING([--with-system-libgltf],
        [Use libgltf already on system.]),,
    [with_system_libgltf="$with_system_libs"])

AC_ARG_WITH(system-clucene,
    AS_HELP_STRING([--with-system-clucene],
        [Use clucene already on system.]),,
    [with_system_clucene="$with_system_libs"])

AC_ARG_WITH(system-expat,
    AS_HELP_STRING([--with-system-expat],
        [Use expat already on system.]),,
    [with_system_expat="$with_system_libs"])

AC_ARG_WITH(system-libxml,
    AS_HELP_STRING([--with-system-libxml],
        [Use libxml/libxslt already on system.]),,
    [with_system_libxml=auto])

AC_ARG_WITH(system-icu,
    AS_HELP_STRING([--with-system-icu],
        [Use icu already on system.]),,
    [with_system_icu="$with_system_libs"])

AC_ARG_WITH(system-ucpp,
    AS_HELP_STRING([--with-system-ucpp],
        [Use ucpp already on system.]),,
    [])

AC_ARG_WITH(system-opencollada,
    AS_HELP_STRING([--with-system-opencollada],
        [Use openCOLLADA already on system.]),,
    [with_system_opencollada=no])

AC_ARG_WITH(system-openldap,
    AS_HELP_STRING([--with-system-openldap],
        [Use the OpenLDAP LDAP SDK already on system.]),,
    [with_system_openldap="$with_system_libs"])

AC_ARG_WITH(system-poppler,
    AS_HELP_STRING([--with-system-poppler],
        [Use system poppler (only needed for PDF import).]),,
    [with_system_poppler="$with_system_libs"])

AC_ARG_WITH(system-apache-commons,
    AS_HELP_STRING([--with-system-apache-commons],
        [Use Apache commons libraries already on system.]),,
    [with_system_apache_commons="$with_system_jars"])

AC_ARG_WITH(system-mariadb,
    AS_HELP_STRING([--with-system-mariadb],
        [Use MariaDB/MySQL libraries already on system, for building the MariaDB Connector/LibreOffice
         extension.]),,
    [with_system_mariadb="$with_system_libs"])

AC_ARG_ENABLE(bundle-mariadb,
    AS_HELP_STRING([--enable-bundle-mariadb],
        [When using MariaDB/MySQL libraries already on system, bundle them with the MariaDB Connector/LibreOffice extension.])
)

AC_ARG_WITH(system-mysql-cppconn,
    AS_HELP_STRING([--with-system-mysql-cppconn],
        [Use MySQL C++ Connector libraries already on system.]),,
    [with_system_mysql_cppconn="$with_system_libs"])

AC_ARG_WITH(system-postgresql,
    AS_HELP_STRING([--with-system-postgresql],
        [Use PostgreSQL libraries already on system, for building the PostgreSQL-SDBC
         driver. If pg_config is not in PATH, use PGCONFIG to point to it.]),,
    [with_system_postgresql="$with_system_libs"])

AC_ARG_WITH(libpq-path,
    AS_HELP_STRING([--with-libpq-path],
        [Use this PostgreSQL C interface (libpq) installation for building
         the PostgreSQL-SDBC extension.])
    [
                          Usage:     --with-libpq-path=<absolute path to
                                                  your libpq installation>
    ],
,)

AC_ARG_WITH(system-firebird,
    AS_HELP_STRING([--with-system-firebird],
        [Use Firebird libraries already on system, for building the Firebird-SDBC
         driver. If fb_config is not in PATH, use FBCONFIG to point to it.]),,
    [with_system_firebird="$with_system_libs"])

AC_ARG_WITH(system-hsqldb,
    AS_HELP_STRING([--with-system-hsqldb],
        [Use hsqldb already on system.]))

AC_ARG_WITH(hsqldb-jar,
    AS_HELP_STRING([--with-hsqldb-jar=JARFILE],
        [Specify path to jarfile manually.]),
    HSQLDB_JAR=$withval)

AC_ARG_ENABLE(scripting-beanshell,
    AS_HELP_STRING([--disable-scripting-beanshell],
        [Disable support for scripts in BeanShell.]),
,
)

AC_ARG_WITH(system-beanshell,
    AS_HELP_STRING([--with-system-beanshell],
        [Use beanshell already on system.]),,
    [with_system_beanshell="$with_system_jars"])

AC_ARG_WITH(beanshell-jar,
    AS_HELP_STRING([--with-beanshell-jar=JARFILE],
        [Specify path to jarfile manually.]),
    BSH_JAR=$withval)

AC_ARG_ENABLE(scripting-javascript,
    AS_HELP_STRING([--disable-scripting-javascript],
        [Disable support for scripts in JavaScript.]),
,
)

AC_ARG_WITH(system-rhino,
    AS_HELP_STRING([--with-system-rhino],
        [Use rhino already on system.]),,)
#    [with_system_rhino="$with_system_jars"])
# Above is not used as we have different debug interface
# patched into internal rhino. This code needs to be fixed
# before we can enable it by default.

AC_ARG_WITH(rhino-jar,
    AS_HELP_STRING([--with-rhino-jar=JARFILE],
        [Specify path to jarfile manually.]),
    RHINO_JAR=$withval)

AC_ARG_WITH(commons-codec-jar,
    AS_HELP_STRING([--with-commons-codec-jar=JARFILE],
        [Specify path to jarfile manually.]),
    COMMONS_CODEC_JAR=$withval)

AC_ARG_WITH(commons-lang-jar,
    AS_HELP_STRING([--with-commons-lang-jar=JARFILE],
        [Specify path to jarfile manually.]),
    COMMONS_LANG_JAR=$withval)

AC_ARG_WITH(commons-httpclient-jar,
    AS_HELP_STRING([--with-commons-httpclient-jar=JARFILE],
        [Specify path to jarfile manually.]),
    COMMONS_HTTPCLIENT_JAR=$withval)

AC_ARG_WITH(commons-logging-jar,
    AS_HELP_STRING([--with-commons-logging-jar=JARFILE],
        [Specify path to jarfile manually.]),
    COMMONS_LOGGING_JAR=$withval)

AC_ARG_WITH(system-jfreereport,
    AS_HELP_STRING([--with-system-jfreereport],
        [Use JFreeReport already on system.]),,
    [with_system_jfreereport="$with_system_jars"])

AC_ARG_WITH(sac-jar,
    AS_HELP_STRING([--with-sac-jar=JARFILE],
        [Specify path to jarfile manually.]),
    SAC_JAR=$withval)

AC_ARG_WITH(libxml-jar,
    AS_HELP_STRING([--with-libxml-jar=JARFILE],
        [Specify path to jarfile manually.]),
    LIBXML_JAR=$withval)

AC_ARG_WITH(flute-jar,
    AS_HELP_STRING([--with-flute-jar=JARFILE],
        [Specify path to jarfile manually.]),
    FLUTE_JAR=$withval)

AC_ARG_WITH(jfreereport-jar,
    AS_HELP_STRING([--with-jfreereport-jar=JARFILE],
        [Specify path to jarfile manually.]),
    JFREEREPORT_JAR=$withval)

AC_ARG_WITH(liblayout-jar,
    AS_HELP_STRING([--with-liblayout-jar=JARFILE],
        [Specify path to jarfile manually.]),
    LIBLAYOUT_JAR=$withval)

AC_ARG_WITH(libloader-jar,
    AS_HELP_STRING([--with-libloader-jar=JARFILE],
        [Specify path to jarfile manually.]),
    LIBLOADER_JAR=$withval)

AC_ARG_WITH(libloader-jar,
    AS_HELP_STRING([--with-libloader-jar=JARFILE],
        [Specify path to jarfile manually.]),
    LIBLOADER_JAR=$withval)

AC_ARG_WITH(libformula-jar,
    AS_HELP_STRING([--with-libformula-jar=JARFILE],
        [Specify path to jarfile manually.]),
    LIBFORMULA_JAR=$withval)

AC_ARG_WITH(librepository-jar,
    AS_HELP_STRING([--with-librepository-jar=JARFILE],
        [Specify path to jarfile manually.]),
    LIBREPOSITORY_JAR=$withval)

AC_ARG_WITH(libfonts-jar,
    AS_HELP_STRING([--with-libfonts-jar=JARFILE],
        [Specify path to jarfile manually.]),
    LIBFONTS_JAR=$withval)

AC_ARG_WITH(libserializer-jar,
    AS_HELP_STRING([--with-libserializer-jar=JARFILE],
        [Specify path to jarfile manually.]),
    LIBSERIALIZER_JAR=$withval)

AC_ARG_WITH(libbase-jar,
    AS_HELP_STRING([--with-libbase-jar=JARFILE],
        [Specify path to jarfile manually.]),
    LIBBASE_JAR=$withval)

AC_ARG_WITH(system-odbc,
    AS_HELP_STRING([--with-system-odbc],
        [Use the odbc headers already on system.]),,
    [with_system_odbc="auto"])

AC_ARG_WITH(system-sane,
    AS_HELP_STRING([--with-system-sane],
        [Use sane.h already on system.]),,
    [with_system_sane="$with_system_headers"])

AC_ARG_WITH(system-bluez,
    AS_HELP_STRING([--with-system-bluez],
        [Use bluetooth.h already on system.]),,
    [with_system_bluez="$with_system_headers"])

AC_ARG_WITH(system-mesa-headers,
    AS_HELP_STRING([--with-system-mesa-headers],
        [Use Mesa headers already on system.]),,
    [with_system_mesa_headers="$with_system_headers"])

AC_ARG_WITH(system-curl,
    AS_HELP_STRING([--with-system-curl],
        [Use curl already on system.]),,
    [with_system_curl=auto])

AC_ARG_WITH(system-boost,
    AS_HELP_STRING([--with-system-boost],
        [Use boost already on system.]),,
    [with_system_boost="$with_system_headers"])

AC_ARG_WITH(system-glm,
    AS_HELP_STRING([--with-system-glm],
        [Use glm already on system.]),,
    [with_system_glm="$with_system_headers"])

AC_ARG_WITH(system-vigra,
    AS_HELP_STRING([--with-system-vigra],
        [Use vigra already on system.]),,
    [with_system_vigra="$with_system_headers"])

AC_ARG_WITH(system-hunspell,
    AS_HELP_STRING([--with-system-hunspell],
        [Use libhunspell already on system.]),,
    [with_system_hunspell="$with_system_libs"])

AC_ARG_WITH(system-mythes,
    AS_HELP_STRING([--with-system-mythes],
        [Use mythes already on system.]),,
    [with_system_mythes="$with_system_libs"])

AC_ARG_WITH(system-altlinuxhyph,
    AS_HELP_STRING([--with-system-altlinuxhyph],
        [Use ALTLinuxhyph already on system.]),,
    [with_system_altlinuxhyph="$with_system_libs"])

AC_ARG_WITH(system-liblangtag,
    AS_HELP_STRING([--with-system-liblangtag],
        [Use liblangtag library already on system.]),,
    [with_system_liblangtag="$with_system_libs"])

AC_ARG_WITH(system-npapi-headers,
    AS_HELP_STRING([--with-system-npapi-headers],
        [Use NPAPI headers provided by system instead of bundled ones. Used in
         extensions/source/plugin (ENABLE_NPAPI_FROM_BROWSER=TRUE)]),,
    [with_system_npapi_headers="$with_system_headers"]
)

AC_ARG_WITH(linker-hash-style,
    AS_HELP_STRING([--with-linker-hash-style],
        [Use linker with --hash-style=<style> when linking shared objects.
         Possible values: "sysv", "gnu", "both". The default value is "gnu"
         if supported on the build system, and "sysv" otherwise.]))

AC_ARG_WITH(jdk-home,
    AS_HELP_STRING([--with-jdk-home],
        [If you have installed JDK 1.3 or later on your system please supply the
         path here. Note that this is not the location of the java command but the
         location of the entire distribution.])
    [
                          Usage:     --with-jdk-home=<absolute path to JDK home>
    ],
,)

AC_ARG_WITH(help,
    AS_HELP_STRING([--with-help],
        [Enable the build of help. There is a special parameter "common" that
         can be used to bundle only the common part, .e.g help-specific icons.
         This is useful when you build the helpcontent separately.])
    [
                          Usage:     --with-help    build the entire local help
                                 --without-help     no local help (default)
                                 --with-help=common bundle common files for the local
                                                    help but do not build the whole help
    ],
,)

AC_ARG_WITH(java,
    AS_HELP_STRING([--with-java],
        [Specify the name of the Java interpreter command. Typically "java"
         which is the default.

         To build without support for Java components, applets, accessibility
         or the XML filters written in Java, use --without-java or --with-java=no.])
    [
                          Usage:     --with-java==<java command>
                                     --without-java
    ],
    [ test -z "$with_java" -o "$with_java" = "yes" && with_java=java ],
    [ with_java=java ]
)

AC_ARG_WITH(jvm-path,
    AS_HELP_STRING([--with-jvm-path],
        [Use a specific JVM search path at runtime.])
    [
                          Usage:     --with-jvm-path=<absolute path to parent of jvm home>

                          e. g.: --with-jvm-path=/usr/lib/
                                 to find JRE/JDK in /usr/lib/jvm/
    ],
,)

AC_ARG_WITH(ant-home,
    AS_HELP_STRING([--with-ant-home],
        [If you have installed Jakarta Ant on your system, please supply the path here.
         Note that this is not the location of the Ant binary but the location
         of the entire distribution.])
    [
                          Usage:     --with-ant-home=<absolute path to Ant home>
    ],
,)

AC_ARG_WITH(export-validation,
    AS_HELP_STRING([--with-export-validation],
        [If you want the exported files to be validated. Right now limited to OOXML files in calc export tests.
         Note: You need an executable script officeotron that takes the path to the file.])
    [
                            Usage: --with-export-validation
    ],
,)

AC_ARG_WITH(junit,
    AS_HELP_STRING([--with-junit],
        [Specifies the JUnit 4 jar file to use for JUnit-based tests.
         --without-junit disables those tests. Not relevant in the --without-java case.])
    [
                          Usage:     --with-junit=<absolute path to JUnit 4 jar>
    ],
,with_junit=yes)

AC_ARG_WITH(hamcrest,
    AS_HELP_STRING([--with-hamcrest],
        [Specifies the hamcrest jar file to use for JUnit-based tests.
         --without-junit disables those tests. Not relevant in the --without-java case.])
    [
                          Usage:     --with-hamcrest=<absolute path to hamcrest jar>
    ],
,with_hamcrest=yes)

AC_ARG_WITH(perl-home,
    AS_HELP_STRING([--with-perl-home],
        [If you have installed Perl 5 Distribution, on your system, please
         supply the path here. Note that this is not the location of the Perl
         binary but the location of the entire distribution.])
    [
                          Usage:     --with-perl-home=<abs. path to Perl 5 home>
    ],
,)

AC_ARG_WITH(
    [doxygen],
    AS_HELP_STRING(
        [--with-doxygen],
        [Specifies the doxygen executable to use when generating ODK C/C++
         documentation. --without-doxygen disables generation of ODK C/C++
         documentation. Not relevant in the --disable-odk case.])
    [
                          Usage:     --with-doxygen=<absolute path to doxygen executable>
    ],,
    [with_doxygen=yes])

AC_ARG_WITH(visual-studio,
    AS_HELP_STRING([--with-visual-studio=<2013/2012>],
        [Specify which Visual Studio version to use in case several are
         are installed. If not specified, the order of preference is
         2013, 2012 (including Express editions). Note that it is
         unclear whether using 2013 actually works as intended.])
    [
                          Usage:     --with-visual-studio=<2013/2012>
    ],
,)

AC_ARG_WITH(windows-sdk,
    AS_HELP_STRING([--with-windows-sdk=<6.0(A)/7.0(A)/7.1(A)/8.0(A)>],
        [Specify which Windows SDK, or "Windows Kit", version to use
         in case the one that came with the selected Visual Studio
         is not what you want for some reason. Note that not all compiler/SDK
         combinations are supported. The intent is that this option should not
         be needed.])
    [
                          Usage:     --with-windows-sdk=6.0(A)/7.0(A)/7.1(A)/8.0(A)>
    ],
,)

AC_ARG_WITH(lang,
    AS_HELP_STRING([--with-lang],
        [Use this option to build LibreOffice with additional UI language support.
         English (US) is always included by default.
         Separate multiple languages with space.
         For all languages, use --with-lang=ALL.])
    [
                          Usage:     --with-lang="es sw tu cs sk"
    ],
,)

AC_ARG_WITH(locales,
    AS_HELP_STRING([--with-locales],
        [Use this option to limit the locale information built in.
         Separate multiple locales with space.
         Very experimental and might well break stuff.
         Just a desperate measure to shrink code and data size.
         By default all the locales available is included.
         This option is completely unrelated to --with-lang.])
    [
                          Affects also our character encoding conversion
                          tables for encodings mainly targeted for a
                          particular locale, like EUC-CN and EUC-TW for
                          zh, ISO-2022-JP for ja.

                          Affects also our add-on break iterator data for
                          some languages.

                          For the default, all locales, don't use this switch at all.
                          Specifying just the language part of a locale means all matching
                          locales will be included.

                          Usage:     --with-locales="en es pt fr zh kr ja"
    ],
,)

# Kerberos and GSSAPI used only by PostgreSQL as of LibO 3.5
AC_ARG_WITH(krb5,
    AS_HELP_STRING([--with-krb5],
        [Enable MIT Kerberos 5 support in modules that support it.
         By default automatically enabled on platforms
         where a good system Kerberos 5 is available.]),
,)

AC_ARG_WITH(gssapi,
    AS_HELP_STRING([--with-gssapi],
        [Enable GSSAPI support in modules that support it.
         By default automatically enabled on platforms
         where a good system GSSAPI is available.]),
,)

dnl ===================================================================
dnl Branding
dnl ===================================================================

AC_ARG_WITH(branding,
    AS_HELP_STRING([--with-branding],
        [Use given path to retrieve branding images set.])
    [
                          Search for intro.png about.svg and flat_logo.svg.
                          If any is missing, default ones will be used instead.

                          Search also progress.conf for progress
                          settings on intro screen :

                          PROGRESSBARCOLOR="255,255,255" Set color of
                          progress bar. Comma separated RGB decimal values.
                          PROGRESSSIZE="407,6" Set size of progress bar.
                          Comma separated decimal values (width, height).
                          PROGRESSPOSITION="61,317" Set position of progress
                          bar from left,top. Comma separated decimal values.
                          PROGRESSFRAMECOLOR="20,136,3" Set color of progress
                          bar frame. Comma separated RGB decimal values.
                          PROGRESSTEXTCOLOR="255,255,255" Set color of progress
                          bar text. Comma separated RGB decimal values.
                          PROGRESSTEXTBASELINE="287" Set vertical position of
                          progress bar text from top. Decimal value.

                          Default values will be used if not found.

                          Usage:     --with-branding=/path/to/images
    ],
,)


AC_ARG_WITH(extra-buildid,
    AS_HELP_STRING([--with-extra-buildid],
        [Show addition build identification in about dialog.])
    [
                          Usage:     --with-extra-buildid="Tinderbox: Win-x86@6, Branch:master, Date:2012-11-26_00.29.34"
    ],
,)


AC_ARG_WITH(vendor,
    AS_HELP_STRING([--with-vendor],
        [Set vendor of the build.])
    [
                          Usage:     --with-vendor="John the Builder"
    ],
,)

AC_ARG_WITH(compat-oowrappers,
    AS_HELP_STRING([--with-compat-oowrappers],
        [Install oo* wrappers in parallel with
         lo* ones to keep backward compatibility.
         Has effect only with make distro-pack-install]),
,)

AC_ARG_WITH(os-version,
    AS_HELP_STRING([--with-os-version],
        [For FreeBSD users, use this option option to override the detected OSVERSION.])
    [
                          Usage:     --with-os-version=<OSVERSION>
    ],
,)

AC_ARG_WITH(mingw-cross-compiler,
    AS_HELP_STRING([--with-mingw-cross-compiler],
        [Specify the MinGW cross-compiler to use.])
    [
                          Usage:     --with-mingw-cross-compiler=<mingw32-g++ command>

                          When building on the ODK on Unix and building unowinreg.dll,
                          specify the MinGW C++ cross-compiler.
    ],
,)

AC_ARG_WITH(idlc-cpp,
    AS_HELP_STRING([--with-idlc-cpp],
        [Specify the C Preprocessor to use for idlc.])
    [
                          Usage:     --with-idlc-cpp=cpp

                          Default is ucpp.
    ]
,)

AC_ARG_WITH(build-version,
    AS_HELP_STRING([--with-build-version],
        [Allows the builder to add a custom version tag that will appear in the
         Help/About box for QA purposes.])
    [
                          Usage:     --with-build-version="Built by Jim"
    ],
    with_build_version=$withval ,
)

AC_ARG_WITH(alloc,
    AS_HELP_STRING([--with-alloc],
        [Define which allocator to build with (choices are oo, system, tcmalloc, jemalloc).
         Note that on FreeBSD/NetBSD system==jemalloc]),
,)

AC_ARG_WITH(sun-templates,
    AS_HELP_STRING([--with-sun-templates],
        [Integrate Sun template packages.]),
,)

AC_ARG_WITH(parallelism,
    AS_HELP_STRING([--with-parallelism],
        [Number of jobs to run simultaneously during build. Parallel builds can
        save a lot of time on multi-cpu machines. Defaults to the number of
        CPUs on the machine, unless you configure --enable-icecream - then to
        10.]),
,)

AC_ARG_WITH(all-tarballs,
    AS_HELP_STRING([--with-all-tarballs],
        [Download all external tarballs unconditionally]))

AC_ARG_WITH(gdrive-client-id,
    AS_HELP_STRING([--with-gdrive-client-id],
        [Provides the client id of the application for OAuth2 authentication
        on Google Drive. If either this or --with-gdrive-client-secret is
        empty, the feature will be disabled]),
)

AC_ARG_WITH(gdrive-client-secret,
    AS_HELP_STRING([--with-gdrive-client-secret],
        [Provides the client secret of the application for OAuth2
        authentication on Google Drive. If either this or
        --with-gdrive-client-id is empty, the feature will be disabled]),
)

AC_ARG_WITH(alfresco-cloud-client-id,
    AS_HELP_STRING([--with-alfresco-cloud-client-id],
        [Provides the client id of the application for OAuth2 authentication
        on Alfresco Cloud. If either this or --with-alfresco-cloud-client-secret is
        empty, the feature will be disabled]),
)

AC_ARG_WITH(alfresco-cloud-client-secret,
    AS_HELP_STRING([--with-alfresco-cloud-client-secret],
        [Provides the client secret of the application for OAuth2
        authentication on Alfresco Cloud. If either this or
        --with-alfresco-cloud-client-id is empty, the feature will be disabled]),
)

dnl ===================================================================
dnl Do we want to use pre-build binary tarball for recompile
dnl ===================================================================

if test "$enable_library_bin_tar" = "yes" ; then
    USE_LIBRARY_BIN_TAR=TRUE
else
    USE_LIBRARY_BIN_TAR=
fi
AC_SUBST(USE_LIBRARY_BIN_TAR)

dnl ===================================================================
dnl Test whether build target is Release Build
dnl ===================================================================
AC_MSG_CHECKING([whether build target is Release Build])
if test "$enable_release_build" = "" -o "$enable_release_build" = "no"; then
    AC_MSG_RESULT([no])
    ENABLE_RELEASE_BUILD=
else
    AC_MSG_RESULT([yes])
    ENABLE_RELEASE_BUILD=TRUE
fi
AC_SUBST(ENABLE_RELEASE_BUILD)

dnl ===================================================================
dnl Test whether to sign Windows Build
dnl ===================================================================
AC_MSG_CHECKING([whether to sign windows build])
if test "$enable_windows_build_signing" = "yes" -a "$_os" = "WINNT" -a "$WITH_MINGW" != "YES" ; then
    AC_MSG_RESULT([yes])
    WINDOWS_BUILD_SIGNING="TRUE"
else
    AC_MSG_RESULT([no])
    WINDOWS_BUILD_SIGNING="FALSE"
fi
AC_SUBST(WINDOWS_BUILD_SIGNING)

dnl ===================================================================
dnl MacOSX build and runtime environment options
dnl ===================================================================

AC_ARG_WITH(macosx-sdk,
    AS_HELP_STRING([--with-macosx-sdk],
        [Use a specific SDK for building.])
    [
                          Usage:     --with-macosx-sdk=<version>

                          e. g.: --with-macosx-sdk=10.8

                          there are 3 options to control the MacOSX build:
                          --with-macosx-sdk (refered as 'sdk' below)
                          --with-macosx-version-min-required (refered as 'min' below)
                          --with-macosx-version-max-allowed (refered as 'max' below)

                          the connection between these value and the default they take is as follow:
                          ( ? means not specified on the command line, s means the SDK version found,
                          constraint: 8 <= x <= y <= z)

                          ==========================================
                           command line      || config result
                          ==========================================
                          min  | max  | sdk  || min  | max  | sdk  |
                          ?    | ?    | ?    || 10.8 | 10.s | 10.s |
                          ?    | ?    | 10.x || 10.8 | 10.x | 10.x |
                          ?    | 10.x | ?    || 10.8 | 10.s | 10.s |
                          ?    | 10.x | 10.y || 10.8 | 10.x | 10.y |
                          10.x | ?    | ?    || 10.x | 10.s | 10.s |
                          10.x | ?    | 10.y || 10.x | 10.y | 10.y |
                          10.x | 10.y | ?    || 10.x | 10.y | 10.y |
                          10.x | 10.y | 10.z || 10.x | 10.y | 10.z |


                          see: http://developer.apple.com/library/mac/#technotes/tn2064/_index.html
                          for a detailled technical explanation of these variables

                          Note: MACOSX_DEPLOYMENT_TARGET will be set to the value of 'min'.
    ],
,)

AC_ARG_WITH(macosx-version-min-required,
    AS_HELP_STRING([--with-macosx-version-min-required],
        [set the minimum OS version needed to run the built LibreOffice])
    [
                          Usage:     --with-macosx-version-min-required=<version>

                          e. g.: --with-macos-version-min-required=10.8
                          see --with-macosx-sdk for more info
    ],
,)

AC_ARG_WITH(macosx-version-max-allowed,
    AS_HELP_STRING([--with-macosx-version-max-allowed],
        [set the maximum allowed OS version the LibreOffice compilation can use APIs from])
    [
                          Usage:     --with-macosx-version-max-allowed=<version>

                          e. g.: --with-macos-version-max-allowed=10.8
                          see --with-macosx-sdk for more info
    ],
,)


dnl ===================================================================
dnl options for stuff used during cross-compilation build
dnl Not quite superseded by --with-build-platform-configure-options.
dnl TODO: check, if the "force" option is still needed anywhere.
dnl ===================================================================

AC_ARG_WITH(system-icu-for-build,
    AS_HELP_STRING([--with-system-icu-for-build=yes/no/force],
        [Use icu already on system for build tools (cross-compilation only).]))


dnl ===================================================================
dnl check for required programs (grep, awk, sed, bash)
dnl ===================================================================

pathmunge ()
{
    if test -n "$1"; then
        if test "$build_os" = "cygwin"; then
            if test "$GNUMAKE_WIN_NATIVE" = "TRUE" ; then
                PathFormat "$1"
                new_path=`cygpath -sm "$formatted_path"`
            else
                PathFormat "$1"
                new_path=`cygpath -u "$formatted_path"`
            fi
        else
            new_path="$1"
        fi
        if ! echo "$LO_PATH" | $EGREP -q "(^|:)$1($|:)"; then
            if test "$2" = "after"; then
                LO_PATH="$LO_PATH${P_SEP}$new_path"
            else
                LO_PATH="$new_path${P_SEP}$LO_PATH"
            fi
        fi
        unset new_path
    fi
}

AC_PROG_AWK
AC_PATH_PROG( AWK, $AWK)
if test -z "$AWK"; then
    AC_MSG_ERROR([install awk to run this script])
fi

AC_PATH_PROG(BASH, bash)
if test -z "$BASH"; then
    AC_MSG_ERROR([bash not found in \$PATH])
fi
AC_SUBST(BASH)

AC_MSG_CHECKING([for GNU or BSD tar])
for a in $GNUTAR gtar gnutar tar /usr/sfw/bin/gtar; do
    $a --version 2> /dev/null | egrep "GNU|bsdtar"  2>&1 > /dev/null
    if test $? -eq 0;  then
        GNUTAR=$a
        break
    fi
done
AC_MSG_RESULT($GNUTAR)
if test -z "$GNUTAR"; then
    AC_MSG_ERROR([not found. install GNU or BSD tar.])
fi
AC_SUBST(GNUTAR)

AC_MSG_CHECKING([for tar's option to strip components])
$GNUTAR --help 2> /dev/null | egrep "bsdtar|strip-components" 2>&1 >/dev/null
if test $? -eq 0; then
    STRIP_COMPONENTS="--strip-components"
else
    $GNUTAR --help 2> /dev/null | egrep "strip-path" 2>&1 >/dev/null
    if test $? -eq 0; then
        STRIP_COMPONENTS="--strip-path"
    else
        STRIP_COMPONENTS="unsupported"
    fi
fi
AC_MSG_RESULT($STRIP_COMPONENTS)
if test x$STRIP_COMPONENTS = xunsupported; then
    AC_MSG_ERROR([you need a tar that is able to strip components.])
fi
AC_SUBST(STRIP_COMPONENTS)

dnl It is useful to have a BUILD_TYPE keyword to distinguish "normal"
dnl desktop OSes from "mobile" ones.

dnl We assume that a non-DESKTOP build type is also a non-NATIVE one.
dnl In other words, that when building for an OS that is not a
dnl "desktop" one but a "mobile" one, we are always cross-compiling.

dnl Note the direction of the implication; there is no assumption that
dnl cross-compiling would imply a non-desktop OS.

if test $_os != iOS -a $_os != Android; then
    BUILD_TYPE="$BUILD_TYPE DESKTOP"
    AC_DEFINE(HAVE_FEATURE_DESKTOP)
    AC_DEFINE(HAVE_FEATURE_MULTIUSER_ENVIRONMENT)
fi

DISABLE_EXPORT=''
if test "$enable_export" != no; then
    BUILD_TYPE="$BUILD_TYPE EXPORT"
else
    DISABLE_EXPORT='TRUE'
    SCPDEFS="$SCPDES -DDISABLE_EXPORT"
fi
AC_SUBST(DISABLE_EXPORT)

# Whether to build "avmedia" functionality or not.

if test -z "$enable_avmedia"; then
    enable_avmedia=yes
fi

if test "$enable_avmedia" = yes; then
    BUILD_TYPE="$BUILD_TYPE AVMEDIA"
    AC_DEFINE(HAVE_FEATURE_AVMEDIA)
else
    SCPDEFS="$SCPDEFS -DDISABLE_AVMEDIA"
fi

# Decide whether to build database connectivity stuff (including
# Base) or not. We probably don't want to on non-desktop OSes.

if test -z "$enable_database_connectivity"; then
    # --disable-database-connectivity is unfinished work in progress
    # and the iOS test app doesn't link if we actually try to use it.
    # if test $_os != iOS -a $_os != Android; then
        enable_database_connectivity=yes
    # fi
fi

if test "$enable_database_connectivity" = yes; then
    BUILD_TYPE="$BUILD_TYPE DBCONNECTIVITY"
    AC_DEFINE(HAVE_FEATURE_DBCONNECTIVITY)
fi

if test -z "$enable_extensions"; then
    # For iOS and Android disable extensions unless specifically overridden with --enable-extensions.
    if test $_os != iOS -a $_os != Android; then
        enable_extensions=yes
    fi
fi

if test "$enable_extensions" = yes; then
    BUILD_TYPE="$BUILD_TYPE EXTENSIONS"
    AC_DEFINE(HAVE_FEATURE_EXTENSIONS)
fi

if test -z "$enable_scripting"; then
    # Disable scripting for iOS unless specifically overridden
    # with --enable-scripting.
    if test $_os != iOS; then
        enable_scripting=yes
    fi
fi

DISABLE_SCRIPTING=''
if test "$enable_scripting" = yes; then
    BUILD_TYPE="$BUILD_TYPE SCRIPTING"
    AC_DEFINE(HAVE_FEATURE_SCRIPTING)
else
    DISABLE_SCRIPTING='TRUE'
    SCPDEFS="$SCPDEFS -DDISABLE_SCRIPTING"
fi

if test $_os = iOS -o $_os = Android; then
    # Disable dynamic_loading always for iOS and Android
    enable_dynamic_loading=no
elif test -z "$enable_dynamic_loading"; then
    # Otherwise enable it unless speficically disabled
    enable_dynamic_loading=yes
fi

DISABLE_DYNLOADING=''
if test "$enable_dynamic_loading" = yes; then
    BUILD_TYPE="$BUILD_TYPE DYNLOADING"
else
    DISABLE_DYNLOADING='TRUE'
    SCPDEFS="$SCPDEFS -DDISABLE_DYNLOADING"
fi
AC_SUBST(DISABLE_DYNLOADING)

# remenber SYSBASE value
AC_SUBST(SYSBASE)

dnl ===================================================================
dnl  Sort out various gallery compilation options
dnl ===================================================================
AC_MSG_CHECKING([how to build and package galleries])
if test -n "${with_galleries}"; then
    if test "$with_galleries" = "build"; then
        WITH_GALLERY_BUILD=TRUE
        AC_MSG_RESULT([build from source images internally])
    elif test "$with_galleries" = "no"; then
        WITH_GALLERY_BUILD=
        AC_MSG_RESULT([disable non-internal gallery build])
    else
        AC_MSG_ERROR([unknown value --with-galleries=$with_galleries])
    fi
else
    if test $_os != iOS -a $_os != Android; then
        WITH_GALLERY_BUILD=TRUE
        AC_MSG_RESULT([internal src images for desktop])
    else
        WITH_GALLERY_BUILD=
        AC_MSG_RESULT([disable src imgage build])
    fi
fi
AC_SUBST(WITH_GALLERY_BUILD)

dnl ===================================================================
dnl  Checks if ccache is available
dnl ===================================================================
if test "$enable_ccache" = "yes" -o \( "$enable_ccache" = "" -a "$enable_icecream" != "yes" \); then
    case "%$CC%$CXX%" in
    # If $CC and/or $CXX already contain "ccache" (possibly suffixed with some verison number etc),
    # assume that's good then
    *%ccache[[-_' ']]*|*/ccache[[-_' ']]*)
        AC_MSG_NOTICE([ccache seems to be included in a pre-defined CC and/or CXX])
        ;;
    *)
        AC_PATH_PROG([CCACHE],[ccache],[not found])
        if test "$CCACHE" = "not found"; then
            CCACHE=""
        else
            # Need to check for ccache version: otherwise prevents
            # caching of the results (like "-x objective-c++" for Mac)
            if test $_os = Darwin -o $_os = iOS; then
                # Check ccache version
                AC_MSG_CHECKING([whether version of ccache is suitable])
                CCACHE_VERSION=`"$CCACHE" -V | "$AWK" '/^ccache version/{print $3}'`
                CCACHE_NUMVER=`echo $CCACHE_VERSION | $AWK -F. '{ print \$1*10000+\$2*100+\$3 }'`
                if test "$CCACHE_VERSION" = "2.4_OOo" -o "$CCACHE_NUMVER" -ge "030100"; then
                    AC_MSG_RESULT([yes, $CCACHE_VERSION])
                else
                    AC_MSG_RESULT([no, $CCACHE_VERSION])
                    CCACHE=""
                fi
            fi
        fi
        ;;
    esac
else
    CCACHE=""
fi

if test "$CCACHE" != ""; then
    ccache_size_msg=$([ccache -s | tail -n 1 | sed 's/^[^0-9]*//' | sed -e 's/\.[0-9]*//'])
    ccache_size=$(echo "$ccache_size_msg" | grep "G" | sed -e 's/G.*$//')
    if test "$ccache_size" = ""; then
        ccache_size=$(echo "$ccache_size_msg" | grep "M" | sed -e 's/\ M.*$//')
        if test "$ccache_size" = ""; then
            ccache_size=0
        fi
        # we could not determine the size or it was less than 1GB -> disable auto-ccache
        if test $ccache_size -lt 1024; then
            CCACHE=""
            AC_MSG_WARN([ccache's cache size is less than 1GB using it is counter-productive: Disabling auto-ccache detection])
            add_warning "ccache's cache size is less than 1GB using it is counter-productive: auto-ccache detection disabled"
        else
            # warn that ccache may be too small for debug build
            AC_MSG_WARN([ccache's cache size is less than 5GB using it may be counter-productive for debug or symbol-enabled build])
            add_warning "ccache's cache size is less than 5GB using it may be counter-productive for debug or symbol-enabled build"
        fi
    else
        if test $ccache_size -lt 5; then
            #warn that ccache may be too small for debug build
            AC_MSG_WARN([ccache's cache size is less than 5GB using it may be counter-productive for debug or symbol-enabled build])
            add_warning "ccache's cache size is less than 5GB using it may be counter-productive for debug or symbol-enabled build"
        fi
    fi
fi

dnl ===================================================================
dnl  Checks for C compiler,
dnl  The check for the C++ compiler is later on.
dnl ===================================================================
if test "$_os" != "WINNT" -a "$WITH_MINGW" != "yes"; then
    GCC_HOME_SET="true"
    AC_MSG_CHECKING([gcc home])
    if test -z "$with_gcc_home"; then
        if test "$enable_icecream" = "yes"; then
            if test -d "/usr/lib/icecc/bin"; then
                GCC_HOME="/usr/lib/icecc/"
            else
                GCC_HOME="/opt/icecream/"
            fi
        else
            GCC_HOME=`which gcc | $SED -e s,/bin/gcc,,`
            GCC_HOME_SET="false"
        fi
    else
        GCC_HOME="$with_gcc_home"
    fi
    AC_MSG_RESULT($GCC_HOME)
    AC_SUBST(GCC_HOME)

    if test "$GCC_HOME_SET" = "true"; then
        if test -z "$CC"; then
            CC="$GCC_HOME/bin/gcc"
        fi
        if test -z "$CXX"; then
            CXX="$GCC_HOME/bin/g++"
        fi
    fi
fi

dnl The following checks for gcc, cc and then cl (if it weren't guarded for win32)
if test "$_os" != "WINNT" -o "$WITH_MINGW" = "yes"; then
    # AC_PROG_CC sets CFLAGS to -g -O2 if not set, avoid that
    save_CFLAGS=$CFLAGS
    AC_PROG_CC
    CFLAGS=$save_CFLAGS
fi

COMPATH=`dirname "$CC"`
if test "$COMPATH" = "."; then
    AC_PATH_PROGS(COMPATH, $CC)
    dnl double square bracket to get single because of M4 quote...
    COMPATH=`echo $COMPATH | $SED "s@/[[^/:]]*\\\$@@"`
fi
COMPATH=`echo $COMPATH | $SED "s@/[[Bb]][[Ii]][[Nn]]\\\$@@"`

AC_MSG_CHECKING([whether to treat the installation as read-only])

if test \( -z "$enable_readonly_installset" -a "$ENABLE_MACOSX_SANDBOX" = TRUE \) -o \
        "$enable_extensions" != yes; then
    enable_readonly_installset=yes
fi
if test "$enable_readonly_installset" = yes; then
    AC_MSG_RESULT([yes])
    AC_DEFINE(HAVE_FEATURE_READONLY_INSTALLSET)
else
    AC_MSG_RESULT([no])
fi

dnl ===================================================================
dnl Structure of install set
dnl ===================================================================

if test $_os = Darwin; then
    LIBO_BIN_FOLDER=MacOS
    LIBO_ETC_FOLDER=Resources
    LIBO_LIBEXEC_FOLDER=MacOS
    LIBO_LIB_FOLDER=Frameworks
    LIBO_SHARE_FOLDER=Resources
    LIBO_SHARE_HELP_FOLDER=Resources/help
    LIBO_SHARE_PRESETS_FOLDER=Resources/presets
    LIBO_LIB_PYUNO_FOLDER=Resources
    LIBO_SHARE_RESOURCE_FOLDER=Resources/resource
    LIBO_SHARE_SHELL_FOLDER=Resources/shell
    LIBO_URE_BIN_FOLDER=MacOS
    LIBO_URE_ETC_FOLDER=Resources/ure/etc
    LIBO_URE_LIB_FOLDER=Frameworks
    LIBO_URE_SHARE_FOLDER=Resources/ure/share
elif test $_os = WINNT; then
    LIBO_BIN_FOLDER=program
    LIBO_ETC_FOLDER=program
    LIBO_LIBEXEC_FOLDER=program
    LIBO_LIB_FOLDER=program
    LIBO_LIB_PYUNO_FOLDER=program
    LIBO_SHARE_FOLDER=share
    LIBO_SHARE_HELP_FOLDER=help
    LIBO_SHARE_JAVA_FOLDER=program/classes
    LIBO_SHARE_PRESETS_FOLDER=presets
    LIBO_SHARE_RESOURCE_FOLDER=program/resource
    LIBO_SHARE_SHELL_FOLDER=program/shell
    LIBO_URE_BIN_FOLDER=URE/bin
    LIBO_URE_ETC_FOLDER=URE/bin
    LIBO_URE_LIB_FOLDER=URE/bin
    LIBO_URE_SHARE_FOLDER=URE
else
    LIBO_BIN_FOLDER=program
    LIBO_ETC_FOLDER=program
    LIBO_LIBEXEC_FOLDER=program
    LIBO_LIB_FOLDER=program
    LIBO_LIB_PYUNO_FOLDER=program
    LIBO_SHARE_FOLDER=share
    LIBO_SHARE_HELP_FOLDER=help
    LIBO_SHARE_JAVA_FOLDER=program/classes
    LIBO_SHARE_PRESETS_FOLDER=presets
    LIBO_SHARE_RESOURCE_FOLDER=program/resource
    LIBO_SHARE_SHELL_FOLDER=program/shell
    LIBO_URE_BIN_FOLDER=ure/bin
    LIBO_URE_ETC_FOLDER=ure/lib
    LIBO_URE_LIB_FOLDER=ure/lib
    LIBO_URE_SHARE_FOLDER=ure/share
fi
AC_DEFINE_UNQUOTED(LIBO_BIN_FOLDER,"$LIBO_BIN_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_ETC_FOLDER,"$LIBO_ETC_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_LIBEXEC_FOLDER,"$LIBO_LIBEXEC_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_LIB_FOLDER,"$LIBO_LIB_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_LIB_PYUNO_FOLDER,"$LIBO_LIB_PYUNO_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_SHARE_FOLDER,"$LIBO_SHARE_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_SHARE_HELP_FOLDER,"$LIBO_SHARE_HELP_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_SHARE_JAVA_FOLDER,"$LIBO_SHARE_JAVA_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_SHARE_PRESETS_FOLDER,"$LIBO_SHARE_PRESETS_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_SHARE_RESOURCE_FOLDER,"$LIBO_SHARE_RESOURCE_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_SHARE_SHELL_FOLDER,"$LIBO_SHARE_SHELL_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_URE_BIN_FOLDER,"$LIBO_URE_BIN_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_URE_ETC_FOLDER,"$LIBO_URE_ETC_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_URE_LIB_FOLDER,"$LIBO_URE_LIB_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_URE_SHARE_FOLDER,"$LIBO_URE_SHARE_FOLDER")
AC_DEFINE_UNQUOTED(LIBO_URE_SHARE_JAVA_FOLDER,"$LIBO_URE_SHARE_JAVA_FOLDER")

# Not all of them needed in config_host.mk, add more if need arises
AC_SUBST(LIBO_BIN_FOLDER)
AC_SUBST(LIBO_ETC_FOLDER)
AC_SUBST(LIBO_LIB_FOLDER)
AC_SUBST(LIBO_LIB_PYUNO_FOLDER)
AC_SUBST(LIBO_SHARE_FOLDER)
AC_SUBST(LIBO_SHARE_HELP_FOLDER)
AC_SUBST(LIBO_SHARE_JAVA_FOLDER)
AC_SUBST(LIBO_SHARE_PRESETS_FOLDER)
AC_SUBST(LIBO_SHARE_RESOURCE_FOLDER)
AC_SUBST(LIBO_URE_BIN_FOLDER)
AC_SUBST(LIBO_URE_ETC_FOLDER)
AC_SUBST(LIBO_URE_LIB_FOLDER)
AC_SUBST(LIBO_URE_SHARE_FOLDER)
AC_SUBST(LIBO_URE_SHARE_JAVA_FOLDER)

dnl ===================================================================
dnl Windows specific tests and stuff
dnl ===================================================================

# Get a value from the 32-bit side of the Registry
reg_get_value_32()
{
    # Return value: $regvalue
    unset regvalue
    _regvalue=`cat "/proc/registry32/$1" 2> /dev/null`

    if test $? -eq 0; then
        regvalue=$_regvalue
    fi
}

# Get a value from the 64-bit side of the Registry
reg_get_value_64()
{
    # Return value: $regvalue
    unset regvalue
    _regvalue=`cat "/proc/registry64/$1" 2> /dev/null`

    if test $? -eq 0; then
        regvalue=$_regvalue
    fi
}

if test "$_os" = "WINNT"; then
    AC_MSG_CHECKING([whether to build a 64-bit LibreOffice])
    if test "$enable_64_bit" = "" -o "$enable_64_bit" = "no"; then
        AC_MSG_RESULT([no])
        WINDOWS_SDK_ARCH="x86"
    else
        AC_MSG_RESULT([yes])
        WINDOWS_SDK_ARCH="x64"
        BITNESS_OVERRIDE=64
    fi
fi

if test "$cross_compiling" = "yes"; then
    export CROSS_COMPILING=TRUE
    SCPDEFS="$SCPDEFS -DCROSS_COMPILING"
else
    CROSS_COMPILING=
    BUILD_TYPE="$BUILD_TYPE NATIVE"
fi
AC_SUBST(CROSS_COMPILING)

dnl ===================================================================
dnl  Is GCC actually Clang?
dnl ===================================================================

COM_GCC_IS_CLANG=
if test "$GCC" = "yes"; then
    AC_MSG_CHECKING([whether GCC is actually Clang])
    AC_COMPILE_IFELSE([AC_LANG_SOURCE([[
        #ifndef __clang__
        you lose
        #endif
        int foo=42;
        ]])],
        [AC_MSG_RESULT([yes])
         COM_GCC_IS_CLANG=TRUE],
        [AC_MSG_RESULT([no])])

    if test "$COM_GCC_IS_CLANG" = TRUE; then
        AC_MSG_CHECKING([the Clang version])
        clang_version=`echo __clang_major__.__clang_minor__.__clang_patchlevel__ | $CC -E -P -`
        CLANG_FULL_VERSION=`echo __clang_version__ | $CC -E -P -`
        CLANGVER=`echo $clang_version | $AWK -F. '{ print \$1*10000+\$2*100+\$3 }'`
        AC_MSG_RESULT([Clang $CLANG_FULL_VERSION, $CLANGVER])
        AC_DEFINE_UNQUOTED(CLANG_FULL_VERSION,$CLANG_FULL_VERSION)
    fi

    # On Windows MSVC only supports C90 so force gnu89 (especially in clang) to
    # to catch potential gnu89/C90 incompatibilities locally.
    CFLAGS="$CFLAGS -std=gnu89"
fi
AC_SUBST(COM_GCC_IS_CLANG)

if test "$CCACHE" != "" -a "$COM_GCC_IS_CLANG" = TRUE; then
    if test -z "$CCACHE_CPP2"; then
        AC_MSG_WARN([Using ccache with Clang without CCACHE_CPP2 set causes spurious warnings.])
        add_warning "Using ccache with Clang without CCACHE_CPP2 set causes spurious warnings."
    fi
fi

dnl ===================================================================
dnl  Test the gcc version
dnl ===================================================================
if test "$GCC" = "yes" -a -z "$COM_GCC_IS_CLANG"; then
    AC_MSG_CHECKING([the GCC version])
    _gcc_version=`$CC -dumpversion`
    GCC_VERSION=`echo $_gcc_version | $AWK -F. '{ print \$1*100+\$2 }'`

    AC_MSG_RESULT([gcc $_gcc_version])

    if test "$GCC_VERSION" -lt 0406; then
        AC_MSG_ERROR([GCC $_gcc_version is too old, must be at least GCC 4.6.0])
    fi
else
    # Explicitly force GCC_VERSION to be empty, even for Clang, to check incorrect uses.
    # GCC version should generally be checked only when handling GCC-specific bugs, for testing
    # things like features configure checks should be used, otherwise they may e.g. fail with Clang
    # (which reports itself as GCC 4.2.1).
    GCC_VERSION=
fi
AC_SUBST(GCC_VERSION)

# ===================================================================
# check various GCC options that Clang does not support now but maybe
# will somewhen in the future, check them even for GCC, so that the
# flags are set
# ===================================================================

HAVE_GCC_GGDB2=
HAVE_GCC_FINLINE_LIMIT=
HAVE_GCC_FNO_INLINE=
if test "$GCC" = "yes"; then
    AC_MSG_CHECKING([whether $CC supports -ggdb2])
    if test -n "$CLANGVER" -a 0"$CLANGVER" -le 30100; then
        # Option just ignored and silly warning that isn't a real
        # warning printed
        :
    else
        save_CFLAGS=$CFLAGS
        CFLAGS="$CFLAGS -Werror -ggdb2"
        AC_LINK_IFELSE([AC_LANG_PROGRAM([[]], [[ return 0; ]])],[ HAVE_GCC_GGDB2=TRUE ],[])
        CFLAGS=$save_CFLAGS
    fi
    if test "$HAVE_GCC_GGDB2" = "TRUE"; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi

    AC_MSG_CHECKING([whether $CC supports -finline-limit=0])
    if test -n "$CLANGVER" -a 0"$CLANGVER" -le 30100; then
        # As above
        :
    else
        save_CFLAGS=$CFLAGS
        CFLAGS="$CFLAGS -Werror -finline-limit=0"
        AC_LINK_IFELSE([AC_LANG_PROGRAM([[]], [[ return 0; ]])],[ HAVE_GCC_FINLINE_LIMIT=TRUE ],[])
        CFLAGS=$save_CFLAGS
    fi
    if test "$HAVE_GCC_FINLINE_LIMIT" = "TRUE"; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi

    AC_MSG_CHECKING([whether $CC supports -fno-inline])
    if test -n "$CLANGVER" -a 0"$CLANGVER" -le 30100; then
        # Ditto
        :
    else
        save_CFLAGS=$CFLAGS
        CFLAGS="$CFLAGS -Werror -fno-inline"
        AC_LINK_IFELSE([AC_LANG_PROGRAM([[]], [[ return 0; ]])],[ HAVE_GCC_FNO_INLINE=TRUE ],[])
        CFLAGS=$save_CFLAGS
    fi
    if test "$HAVE_GCC_FNO_INLINE" = "TRUE"; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_SUBST(HAVE_GCC_GGDB2)
AC_SUBST(HAVE_GCC_FINLINE_LIMIT)
AC_SUBST(HAVE_GCC_FNO_INLINE)

HAVE_LD_BSYMBOLIC_FUNCTIONS=
if test "$GCC" = "yes"; then
    AC_MSG_CHECKING( for -Bsymbolic-functions linker support )
    bsymbolic_functions_ldflags_save=$LDFLAGS
    LDFLAGS="$LDFLAGS -Wl,-Bsymbolic-functions -Wl,--dynamic-list-cpp-new -Wl,--dynamic-list-cpp-typeinfo"
    AC_LINK_IFELSE([AC_LANG_PROGRAM([
#include <stdio.h>
        ],[
printf ("hello world\n");
        ])], HAVE_LD_BSYMBOLIC_FUNCTIONS=TRUE, [])
    if test "$HAVE_LD_BSYMBOLIC_FUNCTIONS" = "TRUE"; then
        AC_MSG_RESULT( found )
    else
        AC_MSG_RESULT( not found )
    fi
    LDFLAGS=$bsymbolic_functions_ldflags_save
fi
AC_SUBST(HAVE_LD_BSYMBOLIC_FUNCTIONS)

# Use -isystem (gcc) if possible, to avoid warnigs in 3rd party headers.
# NOTE: must _not_ be used for bundled external libraries!
ISYSTEM=
if test "$GCC" = "yes"; then
    AC_MSG_CHECKING( for -isystem )
    save_CFLAGS=$CFLAGS
    CFLAGS="$CFLAGS -Werror"
    AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]], [[ return 0; ]])],[ ISYSTEM="-isystem " ],[])
    CFLAGS=$save_CFLAGS
    if test -n "$ISYSTEM"; then
        AC_MSG_RESULT(yes)
    else
        AC_MSG_RESULT(no)
    fi
fi
if test -z "$ISYSTEM"; then
    # fall back to using -I
    ISYSTEM=-I
fi
AC_SUBST(ISYSTEM)

dnl ===================================================================
dnl  Check which Visual Studio or MinGW compiler is used
dnl ===================================================================

map_vs_year_to_version()
{
    # Return value: $vsversion

    unset vsversion

    case $1 in
    2012)
        vsversion=11.0;;
    2013)
        vsversion=12.0;;
    *)
        AC_MSG_ERROR([Assertion failure - invalid argument "$1" to map_vs_year_to_version()]);;
    esac
}

vs_versions_to_check()
{
    # Args: $1 (optional) : versions to check, in the order of preference
    # Return value: $vsversions

    unset vsversions

    if test -n "$1"; then
        map_vs_year_to_version "$1"
        vsversions=$vsversion
    else
        # By default we prefer 2013, then 2012
        vsversions="12.0 11.0"
    fi
}

find_msvs()
{
    # Find Visual Studio 2013/2012
    # Args: $1 (optional) : versions to check, in the order of preference
    # Return value: $vstest

    unset vstest

    vs_versions_to_check "$1"

    for ver in $vsversions; do
        reg_get_value_32 HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/VisualStudio/$ver/Setup/VS/ProductDir
        if test -n "$regvalue"; then
            vstest=$regvalue
            break
        fi
        reg_get_value_32 HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/VCExpress/$ver/Setup/VS/ProductDir
        if test -n "$regvalue"; then
            vstest=$regvalue
            break
        fi
    done
}

find_msvc()
{
    # Find Visual C++ 2013/2012
    # Args: $1 (optional) : The VS version year
    # Return values: $vctest, $vcyear, $vcnum, $vcnumwithdot

    unset vctest vcnum vcnumwithdot vcexpress

    vs_versions_to_check "$1"

    for ver in $vsversions; do
        reg_get_value_32 HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/VisualStudio/$ver/Setup/VC/ProductDir
        if test -n "$regvalue"; then
            vctest=$regvalue
            break
        fi
        reg_get_value_32 HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/VCExpress/$ver/Setup/VC/ProductDir
        if test -n "$regvalue"; then
            vctest=$regvalue
            break
        fi
    done
    if test -n "$vctest"; then
        vcnumwithdot=$ver
        case "$vcnumwithdot" in
        11.0)
            vcyear=2012
            vcnum=110
            ;;
        12.0)
            vcyear=2013
            vcnum=120
            ;;
        esac
        reg_get_value_32 HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/VCExpress/$vcnumwithdot/Setup/VC/ProductDir
        if test -n "$regvalue" -a "$regvalue" = "$vctest" ; then
            vcexpress=Express
        fi
    fi
}

SHOWINCLUDES_PREFIX=
MSBUILD_PATH=
if test "$_os" = "WINNT"; then
    if test "$WITH_MINGW" != "yes"; then
        AC_MSG_CHECKING([Visual C++])

        find_msvc "$with_visual_studio"

        if test -z "$vctest"; then
            if test -n "$with_visual_studio"; then
                AC_MSG_ERROR([No Visual Studio $with_visual_studio installation found])
            else
                AC_MSG_ERROR([No Visual Studio 2012 or 2013 installation found])
            fi
        fi

        if test "$BITNESS_OVERRIDE" = ""; then
            if test -f "$vctest/bin/cl.exe"; then
                VC_PRODUCT_DIR=$vctest
            else
                AC_MSG_ERROR([No compiler (cl.exe) in $vctest/bin/cl.exe])
            fi
        else
            if test -f "$vctest/bin/amd64/cl.exe"; then
                VC_PRODUCT_DIR=$vctest
            else
                AC_MSG_ERROR([No compiler (cl.exe) in $vctest/bin/amd64/cl.exe])
            fi
        fi

        VC_PRODUCT_DIR=`win_short_path_for_make "$VC_PRODUCT_DIR"`
        AC_MSG_RESULT([$VC_PRODUCT_DIR])

        AC_MSG_CHECKING([Is Visual C++ Express])
        if test "$vcexpress" = "Express" ; then
            AC_MSG_RESULT([Yes])
        else
            AC_MSG_RESULT([No])
        fi

        # Find the proper version of MSBuild.exe to use based on the VS version
        reg_get_value_32 HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/MSBuild/$vcnumwithdot/MSBuildOverrideTasksPath
        if test -n "$regvalue" ; then
            MSBUILD_PATH=`win_short_path_for_make "$regvalue"`
        fi

        dnl ===========================================================
        dnl  Check for the corresponding mspdb*.dll
        dnl ===========================================================

        MSPDB_PATH=

        if test "$BITNESS_OVERRIDE" = ""; then
            if test "$vcnum" = "120"; then
                MSPDB_PATH="$VC_PRODUCT_DIR/../VC/bin"
            else
                MSPDB_PATH="$VC_PRODUCT_DIR/../Common7/IDE"
            fi
        else
            MSPDB_PATH="$VC_PRODUCT_DIR/bin/amd64"
        fi

        mspdbnum=$vcnum

        if test ! -e "$MSPDB_PATH/mspdb${mspdbnum}.dll"; then
            AC_MSG_ERROR([No mspdb${mspdbnum}.dll in $MSPDB_PATH, Visual Studio installation broken?])
        fi

        MSPDB_PATH=`cygpath -d "$MSPDB_PATH"`
        MSPDB_PATH=`cygpath -u "$MSPDB_PATH"`

        dnl The path needs to be added before cl is called
        PATH="$MSPDB_PATH:$PATH"

        AC_MSG_CHECKING([cl.exe])

        # Is there really ever a need to pass CC explicitly? Surely we can hope to get all the
        # automagical niceness to work OK? If somebody has some unsupported compiler in some weird
        # location, isn't it likely that lots of other things needs changes, too, and just setting CC
        # is not enough?

        if test -z "$CC"; then
            if test "$BITNESS_OVERRIDE" = ""; then
                if test -f "$VC_PRODUCT_DIR/bin/cl.exe"; then
                    CC="$VC_PRODUCT_DIR/bin/cl.exe"
                fi
            else
                if test -f "$VC_PRODUCT_DIR/bin/amd64/cl.exe"; then
                    CC="$VC_PRODUCT_DIR/bin/amd64/cl.exe"
                fi
            fi

            # This gives us a posix path with 8.3 filename restrictions
            CC=`win_short_path_for_make "$CC"`
        fi

        if test -n "$CC"; then
            # Remove /cl.exe from CC case insensitive
            AC_MSG_RESULT([found Visual C++ $vcyear $vcexpress ($CC)])
            if test "$BITNESS_OVERRIDE" = ""; then
                COMPATH=`echo $CC | $SED -e 's@\/[[Bb]][[Ii]][[Nn]]\/[[cC]][[lL]]\.[[eE]][[xX]][[eE]].*@@' -e 's@^.* @@'`
            else
                if test -n "$VC_PRODUCT_DIR"; then
                    # Huh, why not just an assignment?
                    COMPATH=`echo $VC_PRODUCT_DIR`
                fi
            fi
            if test "$BITNESS_OVERRIDE" = ""; then
                CC="$CC -arch:SSE" # MSVC 2012 default for x86 is -arch:SSE2
            fi
            export INCLUDE=`cygpath -d "$COMPATH/Include"`

            PathFormat "$COMPATH"
            COMPATH="$formatted_path"

            VCVER=$vcnum
            MSVSVER=$vcyear

            # The WINDOWS_SDK_ACCEPTABLE_VERSIONS is mostly an educated guess...  Assuming newer ones
            # are always "better", we list them in reverse chronological order.

            case $vcnum in
            110)
                COMEX=14
                WINDOWS_SDK_ACCEPTABLE_VERSIONS="8.0 7.1A"
                ;;
            120)
                COMEX=15
                WINDOWS_SDK_ACCEPTABLE_VERSIONS="8.1A 8.1 8.0"
                ;;
            esac

            # The expectation is that --with-windows-sdk should not need to be used
            if test -n "$with_windows_sdk"; then
                case " $WINDOWS_SDK_ACCEPTABLE_VERSIONS " in
                *" "$with_windows_sdk" "*)
                    WINDOWS_SDK_ACCEPTABLE_VERSIONS=$with_windows_sdk
                    ;;
                *)
                    AC_MSG_ERROR([Windows SDK $with_windows_sdk is not known to work witn VS $MSVSVER])
                    ;;
                esac
            fi

            # Make AC_COMPILE_IFELSE etc. work (set by AC_PROG_C, which we don't use for MSVC)
            ac_objext=obj
            ac_exeext=exe

        else
            AC_MSG_ERROR([Visual C++ not found after all, huh])
        fi

        dnl We need to guess the prefix of the -showIncludes output, it can be
        dnl localized
        AC_MSG_CHECKING([the dependency generation prefix (cl.exe -showIncludes)])
        echo "#include <stdlib.h>" > conftest.c
        SHOWINCLUDES_PREFIX=`$CC -c -showIncludes conftest.c 2>/dev/null | \
            grep 'stdlib\.h' | head -n1 | sed 's/ [[[:alpha:]]]:.*//'`
        rm -f conftest.c conftest.obj
        if test -z "$SHOWINCLUDES_PREFIX"; then
            AC_MSG_ERROR([cannot determine the -showIncludes prefix])
        else
            AC_MSG_RESULT(["$SHOWINCLUDES_PREFIX"])
        fi

        # Check for 64-bit (cross-)compiler to use to build the 64-bit
        # version of the Explorer extension (and maybe other small
        # bits, too) needed when installing a 32-bit LibreOffice on a
        # 64-bit OS. The 64-bit Explorer extension is a feature that
        # has been present since long in OOo. Don't confuse it with
        # building LibreOffice itself as 64-bit code, which is
        # unfinished work and highly experimental.

        BUILD_X64=
        CXX_X64_BINARY=
        LINK_X64_BINARY=

        if test "$BITNESS_OVERRIDE" = ""; then
            AC_MSG_CHECKING([for a x64 compiler and libraries for 64-bit Explorer extensions])
            if test -f "$VC_PRODUCT_DIR/atlmfc/lib/amd64/atls.lib"; then
                # Prefer native x64 compiler to cross-compiler, in case we are running
                # the build on a 64-bit OS.
                if "$VC_PRODUCT_DIR/bin/amd64/cl.exe" -? </dev/null >/dev/null 2>&1; then
                    BUILD_X64=TRUE
                    CXX_X64_BINARY="$VC_PRODUCT_DIR/bin/amd64/cl.exe"
                    LINK_X64_BINARY="$VC_PRODUCT_DIR/bin/amd64/link.exe"
                elif "$VC_PRODUCT_DIR/bin/x86_amd64/cl.exe" -? </dev/null >/dev/null 2>&1; then
                    BUILD_X64=TRUE
                    CXX_X64_BINARY="$VC_PRODUCT_DIR/bin/x86_amd64/cl.exe"
                    LINK_X64_BINARY="$VC_PRODUCT_DIR/bin/x86_amd64/link.exe"
                fi
            fi
            if test "$BUILD_X64" = TRUE; then
                AC_MSG_RESULT([found])
            else
                AC_MSG_RESULT([not found])
                AC_MSG_WARN([Installation set will not contain 64-bit Explorer extensions])
            fi
        fi
        AC_SUBST(BUILD_X64)

        # These are passed to the environment and then used in gbuild/platform/com_MSC_class.mk
        AC_SUBST(CXX_X64_BINARY)
        AC_SUBST(LINK_X64_BINARY)
    else
        AC_MSG_CHECKING([the compiler is MinGW])
        MACHINE_PREFIX=`$CC -dumpmachine`
        if echo $MACHINE_PREFIX | $GREP -q mingw32; then
            COMPATH=`echo "$COMPATH" | sed -e 's,/bin$,,'`
            AC_MSG_RESULT([yes])
        else
            AC_MSG_ERROR([Compiler is not MinGW.])
        fi
    fi
fi
AC_SUBST(VCVER)
PathFormat "$MSPDB_PATH"
MSPDB_PATH="$formatted_path"
AC_SUBST(SHOWINCLUDES_PREFIX)

if test "$_os" = "WINNT" -a "$WITH_MINGW" != yes; then
    AC_MSG_CHECKING([whether to use DirectX])
    if test "$enable_directx" = "yes" -o "$enable_directx" = ""; then
        ENABLE_DIRECTX="TRUE"
        AC_MSG_RESULT([yes])
    else
        ENABLE_DIRECTX=""
        AC_MSG_RESULT([no])
    fi

    AC_MSG_CHECKING([whether to use ActiveX])
    if test "$enable_activex" = "yes" -o "$enable_activex" = "" -a "$vcexpress" != Express; then
        DISABLE_ACTIVEX=""
        AC_MSG_RESULT([yes])
    else
        DISABLE_ACTIVEX="TRUE"
        AC_MSG_RESULT([no])
    fi

else
    ENABLE_DIRECTX=""
    DISABLE_ACTIVEX="TRUE"
fi

AC_SUBST(ENABLE_DIRECTX)
AC_SUBST(DISABLE_ACTIVEX)

#
# dbghelp.dll
#
if test "$_os" = "WINNT"; then
    BUILD_TYPE="$BUILD_TYPE DBGHELP"
fi

#
# prefix C with ccache if needed
#
if test "$CCACHE" != ""; then
    AC_MSG_CHECKING([whether $CC is already ccached])

    AC_LANG_PUSH([C])
    save_CFLAGS=$CFLAGS
    CFLAGS="$CFLAGS --ccache-skip -O2"
    dnl an empty program will do, we're checking the compiler flags
    AC_COMPILE_IFELSE([AC_LANG_PROGRAM([],[])],
                      [use_ccache=yes], [use_ccache=no])
    if test $use_ccache = yes; then
        AC_MSG_RESULT([yes])
    else
        CC="$CCACHE $CC"
        AC_MSG_RESULT([no])
    fi
    CFLAGS=$save_CFLAGS
    AC_LANG_POP([C])
fi

dnl Set the ENABLE_DBGUTIL variable
dnl ===================================================================
AC_MSG_CHECKING([whether to build with additional debug utilities])
if test -n "$enable_dbgutil" -a "$enable_dbgutil" != "no"; then
    ENABLE_DBGUTIL="TRUE"
    # this is an extra var so it can have different default on different MSVC
    # versions (in case there are version specific problems with it)
    MSVC_USE_DEBUG_RUNTIME="TRUE"

    AC_MSG_RESULT([yes])
    # cppunit and graphite expose STL in public headers
    if test "$with_system_cppunit" = "yes"; then
        AC_MSG_ERROR([--with-system-cppunit conflicts with --enable-dbgutil])
    else
        with_system_cppunit=no
    fi
    if test "$with_system_graphite" = "yes"; then
        AC_MSG_ERROR([--with-system-graphite conflicts with --enable-dbgutil])
    else
        with_system_graphite=no
    fi
    if test "$with_system_mysql_cppconn" = "yes"; then
        AC_MSG_ERROR([--with-system-mysql-cppconn conflicts with --enable-dbgutil])
    else
        with_system_mysql_cppconn=no
    fi
    if test "$with_system_orcus" = "yes"; then
        AC_MSG_ERROR([--with-system-orcus conflicts with --enable-dbgutil])
    else
        with_system_orcus=no
    fi
    if test "$with_system_libcmis" = "yes"; then
        AC_MSG_ERROR([--with-system-libcmis conflicts with --enable-dbgutil])
    else
        with_system_libcmis=no
    fi
    if test "$enable_win_mozab_driver" = "yes"; then
        AC_MSG_ERROR([--enable-win-mozab-driver conflicts with --enable-dbgutil])
    fi
else
    ENABLE_DBGUTIL=""
    MSVC_USE_DEBUG_RUNTIME=""
    AC_MSG_RESULT([no])
fi
AC_SUBST(ENABLE_DBGUTIL)
AC_SUBST(MSVC_USE_DEBUG_RUNTIME)

dnl Set the ENABLE_DEBUG variable.
dnl ===================================================================
AC_MSG_CHECKING([whether to do a debug build])
if test -n "$enable_debug" && test "$enable_debug" != "yes" && test "$enable_debug" != "no"; then
    AC_MSG_ERROR([--enable-debug now accepts only yes or no, use --enable-selective-debuginfo])
fi
if test -n "$ENABLE_DBGUTIL" -a "$enable_debug" = "no"; then
    AC_MSG_ERROR([--disable-debug cannot be used with --enable-dbgutil])
fi

if test -n "$ENABLE_DBGUTIL" -o \( -n "$enable_debug" -a "$enable_debug" != "no" \) ; then
    ENABLE_DEBUG="TRUE"
    ANDROID_DEBUGGABLE='android:debuggable="true"'
    ANDROID_INSTALL_LOCATION='android:installLocation="internalOnly"'
    if test -n "$ENABLE_DBGUTIL" ; then
        AC_MSG_RESULT([yes (dbgutil)])
    else
        AC_MSG_RESULT([yes])
    fi
else
    ENABLE_DEBUG=""
    ANDROID_DEBUGGABLE=""
    ANDROID_INSTALL_LOCATION='android:installLocation="preferExternal"'
    AC_MSG_RESULT([no])
fi
AC_SUBST(ENABLE_DEBUG)
AC_SUBST(ANDROID_DEBUGGABLE)
AC_SUBST(ANDROID_INSTALL_LOCATION)

if test "$enable_sal_log" = yes; then
    ENABLE_SAL_LOG=TRUE
fi
AC_SUBST(ENABLE_SAL_LOG)

dnl Selective debuginfo
ENABLE_DEBUGINFO_FOR=
if test -n "$ENABLE_DEBUG"; then
    AC_MSG_CHECKING([whether to use selective debuginfo])
    if test -n "$enable_selective_debuginfo" -a "$enable_selective_debuginfo" != "no"; then
        ENABLE_DEBUGINFO_FOR="$enable_selective_debuginfo"
        AC_MSG_RESULT([for "$enable_selective_debuginfo"])
    else
        ENABLE_DEBUGINFO_FOR=all
        AC_MSG_RESULT([no, for all])
    fi
fi
AC_SUBST(ENABLE_DEBUGINFO_FOR)

dnl Check for enable symbols option
dnl ===================================================================
AC_MSG_CHECKING([whether to include symbols while preserve optimization])
if test -n "$enable_symbols" -a "$enable_symbols" != "no"; then
    ENABLE_SYMBOLS="TRUE"
    if test -n "$ENABLE_DBGUTIL"; then
        AC_MSG_ERROR([--enable-dbgutil cannot be used with --enable-symbols])
    elif test -n "$ENABLE_DEBUG"; then
        AC_MSG_ERROR([--enable-debug cannot be used with --enable-symbols])
    fi
    AC_MSG_RESULT([yes])
else
    if test -n "$enable_symbols" -a "$enable_symbols" = "no"; then
        ENABLE_SYMBOLS="FALSE"
    else
        ENABLE_SYMBOLS=
    fi
    AC_MSG_RESULT([no])
fi
AC_SUBST(ENABLE_SYMBOLS)

# Debug information format for iOS. Running dsymutil takes a long time... you really need a separate
# .dSYM only if running Instruments, I think. (Not for normal debugging in Xcode.) To enable a
# separate .dSYM, either use --enable-release-build or change manually to "DWARF with DSYM" in Xcode.
XCODE_DEBUG_INFORMATION_FORMAT=dwarf-with-dsym
if test "$enable_release_build" != yes -a \( -n "$ENABLE_SYMBOLS" -o -n "$ENABLE_DEBUG" -o -n "$ENABLE_DBGUTIL" \); then
    XCODE_DEBUG_INFORMATION_FORMAT=dwarf
fi
AC_SUBST(XCODE_DEBUG_INFORMATION_FORMAT)

# Check for explicit A/C/CXX/OBJC/OBJCXX/LDFLAGS.
# By default use the ones specified by our build system,
# but explicit override is possible.
AC_MSG_CHECKING(for explicit CFLAGS)
if test -n "$CFLAGS"; then
    AC_MSG_RESULT([$CFLAGS])
    x_CFLAGS=
else
    AC_MSG_RESULT(no)
    x_CFLAGS=[\#]
fi
AC_MSG_CHECKING(for explicit CXXFLAGS)
if test -n "$CXXFLAGS"; then
    AC_MSG_RESULT([$CXXFLAGS])
    x_CXXFLAGS=
else
    AC_MSG_RESULT(no)
    x_CXXFLAGS=[\#]
fi
AC_MSG_CHECKING(for explicit LDFLAGS)
if test -n "$LDFLAGS"; then
    AC_MSG_RESULT([$LDFLAGS])
    x_LDFLAGS=
else
    AC_MSG_RESULT(no)
    x_LDFLAGS=[\#]
fi
AC_SUBST(CFLAGS)
AC_SUBST(CXXFLAGS)
AC_SUBST(LDFLAGS)
AC_SUBST(x_CFLAGS)
AC_SUBST(x_CXXFLAGS)
AC_SUBST(x_LDFLAGS)

#
# determine CPUNAME, GUIBASE, ...
#
SOLARINC=

case "$host_os" in

aix*)
    COM=GCC
    CPUNAME=POWERPC
    GUIBASE=unx
    OS=AIX
    RTL_OS=AIX
    RTL_ARCH=PowerPC
    PLATFORMID=aix_powerpc
    P_SEP=:
    ;;

cygwin*)
    COM=MSC
    GUIBASE=not-used
    OS=WNT
    RTL_OS=Windows
    if test "$GNUMAKE_WIN_NATIVE" = "TRUE" ; then
        P_SEP=";"
    else
        P_SEP=:
    fi
    case "$host_cpu" in
    i*86|x86_64)
        if test "$BITNESS_OVERRIDE" = 64; then
            CPUNAME=X86_64
            RTL_ARCH=X86_64
            PLATFORMID=windows_x86_64
        else
            CPUNAME=INTEL
            RTL_ARCH=x86
            PLATFORMID=windows_x86
        fi
        ;;
    *)
        AC_MSG_ERROR([Unsupported host_cpu $host_cpu for host_os $host_os])
        ;;
    esac
    SCPDEFS="$SCPDEFS -D_MSC_VER"
    ;;

darwin*)
    COM=GCC
    GUIBASE=not-used
    OS=MACOSX
    RTL_OS=MacOSX
    P_SEP=:

    case "$host_cpu" in
    arm*)
        CPUNAME=ARM
        RTL_ARCH=ARM_EABI
        PLATFORMID=ios_arm
        OS=IOS
        ;;
    i*86)
        AC_MSG_ERROR([Can't build 64-bit code in 32-bit OS])
        ;;
    x86_64)
        CPUNAME=X86_64
        RTL_ARCH=X86_64
        PLATFORMID=macosx_x86_64
        ;;
    *)
        AC_MSG_ERROR([Unsupported host_cpu $host_cpu for host_os $host_os])
        ;;
    esac
    ;;

dragonfly*)
    COM=GCC
    GUIBASE=unx
    OS=DRAGONFLY
    RTL_OS=DragonFly
    P_SEP=:

    case "$host_cpu" in
    i*86)
        CPUNAME=INTEL
        RTL_ARCH=x86
        PLATFORMID=dragonfly_x86
        ;;
    x86_64)
        CPUNAME=X86_64
        RTL_ARCH=X86_64
        PLATFORMID=dragonfly_x86_64
        ;;
    *)
        AC_MSG_ERROR([Unsupported host_cpu $host_cpu for host_os $host_os])
        ;;
    esac
    ;;

freebsd*)
    COM=GCC
    GUIBASE=unx
    RTL_OS=FreeBSD
    OS=FREEBSD
    P_SEP=:

    case "$host_cpu" in
    i*86)
        CPUNAME=INTEL
        RTL_ARCH=x86
        PLATFORMID=freebsd_x86
        ;;
    x86_64|amd64)
        CPUNAME=X86_64
        RTL_ARCH=X86_64
        PLATFORMID=freebsd_x86_64
        ;;
    *)
        AC_MSG_ERROR([Unsupported host_cpu $host_cpu for host_os $host_os])
        ;;
    esac
    ;;

kfreebsd*)
    COM=GCC
    GUIBASE=unx
    OS=LINUX
    RTL_OS=kFreeBSD
    P_SEP=:

    case "$host_cpu" in

    i*86)
        CPUNAME=INTEL
        RTL_ARCH=x86
        PLATFORMID=kfreebsd_x86
        ;;
    x86_64)
        CPUNAME=X86_64
        RTL_ARCH=X86_64
        PLATFORMID=kfreebsd_x86_64
        ;;
    *)
        AC_MSG_ERROR([Unsupported host_cpu $host_cpu for host_os $host_os])
        ;;
    esac
    ;;

linux-gnu*)
    COM=GCC
    GUIBASE=unx
    OS=LINUX
    RTL_OS=Linux
    P_SEP=:

    case "$host_cpu" in

    aarch64)
        CPUNAME=AARCH64
        PLATFORMID=linux_aarch64
        RTL_ARCH=AARCH64
        ;;
    alpha)
        CPUNAME=AXP
        RTL_ARCH=ALPHA
        PLATFORMID=linux_alpha
        ;;
    arm*)
        CPUNAME=ARM
        EPM_FLAGS="-a arm"
        RTL_ARCH=ARM_EABI
        PLATFORMID=linux_arm_eabi
        case "$host_cpu" in
        arm*-linux)
            RTL_ARCH=ARM_OABI
            PLATFORMID=linux_arm_oabi
            ;;
        esac
        ;;
    hppa)
        CPUNAME=HPPA
        RTL_ARCH=HPPA
        EPM_FLAGS="-a hppa"
        PLATFORMID=linux_hppa
        ;;
    i*86)
        CPUNAME=INTEL
        RTL_ARCH=x86
        PLATFORMID=linux_x86
        ;;
    ia64)
        CPUNAME=IA64
        RTL_ARCH=IA64
        PLATFORMID=linux_ia64
        ;;
    mips)
        CPUNAME=GODSON
        RTL_ARCH=MIPS_EB
        EPM_FLAGS="-a mips"
        PLATFORMID=linux_mips_eb
        ;;
    mips64)
        CPUNAME=GODSON
        RTL_ARCH=MIPS_EB
        EPM_FLAGS="-a mips64"
        PLATFORMID=linux_mips_eb
        ;;
    mips64el)
        CPUNAME=GODSON
        RTL_ARCH=MIPS_EL
        EPM_FLAGS="-a mips64el"
        PLATFORMID=linux_mips_el
        ;;
    mipsel)
        CPUNAME=GODSON
        RTL_ARCH=MIPS_EL
        EPM_FLAGS="-a mipsel"
        PLATFORMID=linux_mips_el
        ;;
    m68k)
        CPUNAME=M68K
        RTL_ARCH=M68K
        PLATFORMID=linux_m68k
        ;;
    powerpc)
        CPUNAME=POWERPC
        RTL_ARCH=PowerPC
        PLATFORMID=linux_powerpc
        ;;
    powerpc64)
        CPUNAME=POWERPC64
        RTL_ARCH=PowerPC_64
        PLATFORMID=linux_powerpc64
        ;;
    powerpc64le)
        CPUNAME=POWERPC64
        RTL_ARCH=PowerPC_64_LE
        PLATFORMID=linux_powerpc64_le
        ;;
    sparc)
        CPUNAME=SPARC
        RTL_ARCH=SPARC
        PLATFORMID=linux_sparc
        ;;
    s390)
        CPUNAME=S390
        RTL_ARCH=S390
        PLATFORMID=linux_s390
        ;;
    s390x)
        CPUNAME=S390X
        RTL_ARCH=S390x
        PLATFORMID=linux_s390x
        ;;
    x86_64)
        CPUNAME=X86_64
        RTL_ARCH=X86_64
        PLATFORMID=linux_x86_64
        ;;
    *)
        AC_MSG_ERROR([Unsupported host_cpu $host_cpu for host_os $host_os])
        ;;
    esac
    ;;

linux-android*)
    COM=GCC
    GUIBASE=not-used
    OS=ANDROID
    RTL_OS=Android
    P_SEP=:

    case "$host_cpu" in

    arm|armel)
        CPUNAME=ARM
        RTL_ARCH=ARM_EABI
        PLATFORMID=android_arm_eabi
        ;;
    aarch64)
        CPUNAME=AARCH64
        RTL_ARCH=AARCH64
        PLATFORMID=android_aarch64
        ;;
    mips|mipsel)
        CPUNAME=GODSON # Weird, but maybe that's the LO convention?
        RTL_ARCH=MIPS_EL
        PLATFORMID=android_mips_el
        ;;
    i*86)
        CPUNAME=INTEL
        RTL_ARCH=x86
        PLATFORMID=android_x86
        ;;
    *)
        AC_MSG_ERROR([Unsupported host_cpu $host_cpu for host_os $host_os])
        ;;
    esac
    ;;

mingw*)
    COM=GCC
    GUIBASE=not-used
    OS=WNT
    RTL_OS=Windows
    P_SEP=:

    case "$host_cpu" in
    i*86|x86_64)
        if test "$BITNESS_OVERRIDE" = 64; then
            CPUNAME=X86_64
            RTL_ARCH=X86_84
            PLATFORMID=windows_x86_64
            SOLARINC="$SOLARINC -I$SRC_ROOT/include/wntgccx"
        else
            CPUNAME=INTEL
            RTL_ARCH=x86
            PLATFORMID=windows_x86
            SOLARINC="$SOLARINC -I$SRC_ROOT/include/wntgcci"
        fi
        ;;
    *)
        AC_MSG_ERROR([Unsupported host_cpu $host_cpu for host_os $host_os])
        ;;
    esac
    ;;

*netbsd*)

    COM=GCC
    GUIBASE=unx
    OS=NETBSD
    RTL_OS=NetBSD
    P_SEP=:

    case "$host_cpu" in
    i*86)
        CPUNAME=INTEL
        RTL_ARCH=x86
        PLATFORMID=netbsd_x86
        ;;
    powerpc)
        CPUNAME=POWERPC
        RTL_ARCH=PowerPC
        PLATFORMID=netbsd_powerpc
        ;;
    sparc)
        CPUNAME=SPARC
        RTL_ARCH=SPARC
        PLATFORMID=netbsd_sparc
        ;;
    x86_64)
        CPUNAME=X86_64
        RTL_ARCH=X86_64
        PLATFORMID=netbsd_x86_64
        ;;
    *)
        AC_MSG_ERROR([Unsupported host_cpu $host_cpu for host_os $host_os])
        ;;
    esac
    ;;

openbsd*)
    COM=GCC
    GUIBASE=unx
    OS=OPENBSD
    RTL_OS=OpenBSD
    P_SEP=:

    case "$host_cpu" in
    i*86)
        CPUNAME=INTEL
        RTL_ARCH=x86
        PLATFORMID=openbsd_x86
        ;;
    x86_64)
        CPUNAME=X86_64
        RTL_ARCH=X86_64
        PLATFORMID=openbsd_x86_64
        ;;
    *)
        AC_MSG_ERROR([Unsupported host_cpu $host_cpu for host_os $host_os])
        ;;
    esac
    SOLARINC="$SOLARINC -I/usr/local/include"
    ;;

solaris*)

    COM=GCC
    GUIBASE=unx
    OS=SOLARIS
    RTL_OS=Solaris
    P_SEP=:

    case "$host_cpu" in
    i*86)
        CPUNAME=INTEL
        RTL_ARCH=x86
        PLATFORMID=solaris_x86
        ;;
    sparc)
        CPUNAME=SPARC
        RTL_ARCH=SPARC
        PLATFORMID=solaris_sparc
        ;;
    sparc64)
        CPUNAME=SPARC64
        RTL_ARCH=SPARC64
        PLATFORMID=solaris_sparc64
        ;;
    *)
        AC_MSG_ERROR([Unsupported host_cpu $host_cpu for host_os $host_os])
        ;;
    esac
    SOLARINC="$SOLARINC -I/usr/local/include"
    ;;

*)
    AC_MSG_ERROR([$host_os operating system is not suitable to build LibreOffice for!])
    ;;
esac

if test "$enable_headless" = "yes"; then
    if test "$GUIBASE" != "unx"; then
        AC_MSG_ERROR([$host_os operating system is not suitable to build LibreOffice with --enable-headless])
    fi
    GUIBASE=not-used
fi

WORKDIR="${BUILDDIR}/workdir"
INSTDIR="${BUILDDIR}/instdir"
INSTROOT="${INSTDIR}${INSTROOTSUFFIX}"
SOLARINC="-I. -I$SRC_ROOT/include $SOLARINC"
AC_SUBST(COM)
AC_SUBST(CPUNAME)
AC_SUBST(RTL_OS)
AC_SUBST(RTL_ARCH)
AC_SUBST(EPM_FLAGS)
AC_SUBST(GUIBASE)
AC_SUBST([INSTDIR])
AC_SUBST([INSTROOT])
AC_SUBST(OS)
AC_SUBST(P_SEP)
AC_SUBST(WORKDIR)
AC_SUBST(PLATFORMID)
AC_DEFINE_UNQUOTED(WORKDIR,"$WORKDIR")

dnl ===================================================================
dnl Test which package format to use
dnl ===================================================================
AC_MSG_CHECKING([which package format to use])
if test -n "$with_package_format" -a "$with_package_format" != no; then
    for i in $with_package_format; do
        case "$i" in
        aix | bsd | deb | pkg | rpm | archive | dmg | installed | msi)
            ;;
        *)
            AC_MSG_ERROR([unsupported format $i. Supported by EPM are:
aix - AIX software distribution
bsd - FreeBSD, NetBSD, or OpenBSD software distribution
deb - Debian software distribution
pkg - Solaris software distribution
rpm - RedHat software distribution

LibreOffice additionally supports:
archive - .tar.gz or .zip
dmg - Mac OS X .dmg
installed - installation tree
msi - Windows .msi
        ])
            ;;
        esac
    done
    PKGFORMAT="$with_package_format"
    AC_MSG_RESULT([$PKGFORMAT])
else
    PKGFORMAT=
    AC_MSG_RESULT([none])
fi
AC_SUBST(PKGFORMAT)

dnl ===================================================================
dnl Set up a different compiler to produce tools to run on the build
dnl machine when doing cross-compilation
dnl ===================================================================

m4_pattern_allow([PKG_CONFIG_FOR_BUILD])
if test "$cross_compiling" = "yes"; then
    AC_MSG_CHECKING([for BUILD platform configuration])
    echo
    rm -rf CONF-FOR-BUILD config_build.mk
    mkdir CONF-FOR-BUILD
    # Here must be listed all files needed when running the configure script. In particular, also
    # those expanded by the AC_CONFIG_FILES() call near the end of this configure.ac. For clarity,
    # keep them in the same order as there.
    (cd $SRC_ROOT && tar cf - \
        config.guess \
        bin/get_config_variables \
        solenv/bin/getcompver.awk \
        solenv/inc/langlist.mk \
        config_host.mk.in \
        Makefile.in) \
    | (cd CONF-FOR-BUILD && tar xf -)
    cp configure CONF-FOR-BUILD
    test -d config_build && cp -p config_build/*.h CONF-FOR-BUILD/config_host 2>/dev/null
    (
    unset COM GUIBASE OS CPUNAME
    unset CC CXX SYSBASE CFLAGS
    unset AR NM OBJDUMP PKG_CONFIG RANLIB STRIP
    unset LIBXML_CFLAGS LIBXML_LIBS LIBXSLT_CFLAGS LIBXSLT_LIBS XSLTPROC
    test -n "$CC_FOR_BUILD" && export CC="$CC_FOR_BUILD"
    test -n "$CXX_FOR_BUILD" && export CXX="$CXX_FOR_BUILD"
    test -n "$PKG_CONFIG_FOR_BUILD" && export PKG_CONFIG="$PKG_CONFIG_FOR_BUILD"
    cd CONF-FOR-BUILD
    sub_conf_opts=""
    test -n "$enable_ccache" && sub_conf_opts="$sub_conf_opts --enable-ccache=$enable_ccache"
    test -n "$with_ant_home" && sub_conf_opts="$sub_conf_opts --with-ant-home=$with_ant_home"
    test $with_junit = no && sub_conf_opts="$sub_conf_opts --without-junit"
    test -n "$TARFILE_LOCATION" && sub_conf_opts="$sub_conf_opts --with-external-tar=$TARFILE_LOCATION"
    test "$with_system_icu_for_build" = "yes" -o "$with_system_icu_for_build" = "force" && sub_conf_opts="$sub_conf_opts --with-system-icu"
    # we need the msi build tools on mingw if we are creating the
    # installation set
    if test "$WITH_MINGW" = "yes"; then
        enable_winegcc_for_build=
        for pkgformat in $PKGFORMAT; do
            case "$pkgformat" in
            msi|native) enable_winegcc_for_build=yes ;;
            esac
        done
        test -n "$enable_winegcc_for_build" && sub_conf_opts="$sub_conf_opts --enable-winegcc"
    fi
    sub_conf_opts="$sub_conf_opts $with_build_platform_configure_options"
    # Don't bother having configure look for stuff not needed for the build platform anyway
    ./configure \
        --disable-cups \
        --disable-graphite \
        --disable-pdfimport \
        --disable-postgresql-sdbc \
        --with-parallelism="$with_parallelism" \
        --without-doxygen \
        $sub_conf_opts \
        --srcdir=$srcdir \
        2>&1 | sed -e 's/^/    /'
    test -f ./config_host.mk 2>/dev/null || exit
    cp config_host.mk ../config_build.mk
    mv config.log ../config.Build.log
    mkdir -p ../config_build
    . ./bin/get_config_variables CC CXX INSTDIR INSTROOT LIBO_BIN_FOLDER LIBO_LIB_FOLDER LIBO_URE_LIB_FOLDER OS PATH SDKDIRNAME SYSTEM_LIBXML SYSTEM_LIBXSLT WORKDIR

    for V in CC CXX LIBO_BIN_FOLDER LIBO_LIB_FOLDER LIBO_URE_LIB_FOLDER OS SDKDIRNAME SYSTEM_LIBXML SYSTEM_LIBXSLT; do
        VV='$'$V
        VV=`eval "echo $VV"`
        if test -n "$VV"; then
            line=${V}_FOR_BUILD='${'${V}_FOR_BUILD:-$VV'}'
            echo "$line" >>build-config
        fi
    done

    for V in INSTDIR INSTROOT WORKDIR; do
        VV='$'$V
        VV=`eval "echo $VV"`
        VV=`echo $VV | sed -e "s,/CONF-FOR-BUILD/\([[a-z]]*\),/\1_for_build,g"`
        if test -n "$VV"; then
            line="${V}_FOR_BUILD='$VV'"
            echo "$line" >>build-config
        fi
    done

    line=`echo "LO_PATH_FOR_BUILD=$PATH" | sed -e 's,/CONF-FOR-BUILD,,g'`
    echo "$line" >>build-config

    )
    test -f CONF-FOR-BUILD/build-config || AC_MSG_ERROR([Running configure script for BUILD system failed, see CONF-FOR-BUILD/config.log])
    test -f config_build.mk || AC_MSG_ERROR([A file called config_build.mk was supposed to have been copied here, but it isn't found])
    perl -pi -e 's,/(workdir|instdir)(/|$),/\1_for_build\2,g;' \
             -e 's,/CONF-FOR-BUILD,,g;' config_build.mk

    eval `cat CONF-FOR-BUILD/build-config`

    AC_MSG_RESULT([checking for BUILD platform configuration... done])

    rm -rf CONF-FOR-BUILD
else
    OS_FOR_BUILD="$OS"
    CC_FOR_BUILD="$CC"
    CXX_FOR_BUILD="$CXX"
    INSTDIR_FOR_BUILD="$INSTDIR"
    INSTROOT_FOR_BUILD="$INSTROOT"
    LIBO_BIN_FOLDER_FOR_BUILD="$LIBO_BIN_FOLDER"
    LIBO_LIB_FOLDER_FOR_BUILD="$LIBO_LIB_FOLDER"
    LIBO_URE_LIB_FOLDER_FOR_BUILD="$LIBO_URE_LIB_FOLDER"
    SDKDIRNAME_FOR_BUILD="$SDKDIRNAME"
    WORKDIR_FOR_BUILD="$WORKDIR"
fi
AC_SUBST(OS_FOR_BUILD)
AC_SUBST(INSTDIR_FOR_BUILD)
AC_SUBST(INSTROOT_FOR_BUILD)
AC_SUBST(LIBO_BIN_FOLDER_FOR_BUILD)
AC_SUBST(LIBO_LIB_FOLDER_FOR_BUILD)
AC_SUBST(LIBO_URE_LIB_FOLDER_FOR_BUILD)
AC_SUBST(SDKDIRNAME_FOR_BUILD)
AC_SUBST(WORKDIR_FOR_BUILD)

dnl Set the ENABLE_WERROR variable. (Activate --enable-werror)
dnl ===================================================================
AC_MSG_CHECKING([whether to turn warnings to errors])
if test -n "$enable_werror" -a "$enable_werror" != "no"; then
    ENABLE_WERROR="TRUE"
    AC_MSG_RESULT([yes])
else
    AC_MSG_RESULT([no])
fi
AC_SUBST(ENABLE_WERROR)

dnl ===================================================================
AC_MSG_CHECKING([whether to enable pch feature])
if test -n "$enable_pch" && test "$enable_pch" != "no"; then
    if test "$_os" = "WINNT" -a "$WITH_MINGW" != "yes"; then
        ENABLE_PCH="TRUE"
        AC_MSG_RESULT([yes])
    elif test "$GCC" = "yes"; then
        ENABLE_PCH="TRUE"
        AC_MSG_RESULT([yes])
    else
        ENABLE_PCH=""
        AC_MSG_ERROR([Precompiled header not yet supported for your platform/compiler])
    fi
else
    ENABLE_PCH=""
    AC_MSG_RESULT([no])
fi
AC_SUBST(ENABLE_PCH)

TAB=`printf '\t'`

AC_MSG_CHECKING([the GNU Make version])
_make_version=`$GNUMAKE --version | grep GNU | $GREP -v GPL | $SED -e 's@^[[^0-9]]*@@' -e 's@ .*@@' -e 's@,.*@@'`
_make_longver=`echo $_make_version | $AWK -F. '{ print \$1*10000+\$2*100+\$3 }'`
if test "$_make_longver" -ge "038200"; then
    AC_MSG_RESULT([$GNUMAKE $_make_version])

elif test "$_make_longver" -ge "038100"; then
    if test "$build_os" = "cygwin"; then
        AC_MSG_ERROR([failed ($GNUMAKE version >= 3.82 needed])
    fi
    AC_MSG_RESULT([$GNUMAKE $_make_version])

    dnl ===================================================================
    dnl Search all the common names for sha1sum
    dnl ===================================================================
    AC_CHECK_PROGS(SHA1SUM, sha1sum sha1 shasum openssl)
    if test -z "$SHA1SUM"; then
        AC_MSG_ERROR([install the approproate SHA-1 checksumming program for this OS])
    elif test "$SHA1SUM" = "openssl"; then
        SHA1SUM="openssl sha1"
    fi
    AC_MSG_CHECKING([for GNU Make bug 20033])
    TESTGMAKEBUG20033=`mktemp -d tmp.XXXXXX`
    $SED -e "s/<TAB>/$TAB/g" > $TESTGMAKEBUG20033/Makefile << EOF
A := \$(wildcard *.a)

.PHONY: all
all: \$(A:.a=.b)
<TAB>@echo survived bug20033.

.PHONY: setup
setup:
<TAB>@touch 1.a 2.a 3.a 4.a 5.a 6.a

define d1
@echo lala \$(1)
@sleep 1
endef

define d2
@echo tyty \$(1)
@sleep 1
endef

%.b : %.a
<TAB>\$(eval CHECKSUM := \$(word 1,\$(shell cat \$^ | $SHA1SUM))) \$(if \$(wildcard \$(CACHEDIR)/\$(CHECKSUM)),\
<TAB>\$(call d1,\$(CHECKSUM)),\
<TAB>\$(call d2,\$(CHECKSUM)))
EOF
    if test -z "`(cd $TESTGMAKEBUG20033 && $GNUMAKE setup && $GNUMAKE -j)|grep survived`"; then
        no_parallelism_make="YES"
        AC_MSG_RESULT([yes, disable parallelism])
    else
        AC_MSG_RESULT([no, keep parallelism enabled])
    fi
    rm -rf $TESTGMAKEBUG20033
else
    AC_MSG_ERROR([failed ($GNUMAKE version >= 3.81 needed])
fi

# find if gnumake support file function
AC_MSG_CHECKING([whether GNU Make supports the 'file' function])
TESTGMAKEFILEFUNC="`mktemp -d -t tst.XXXXXX`"
if test "$GNUMAKE_WIN_NATIVE" = "TRUE" ; then
    TESTGMAKEFILEFUNC=`cygpath -m $TESTGMAKEFILEFUNC`
fi
$SED -e "s/<TAB>/$TAB/" > $TESTGMAKEFILEFUNC/Makefile << EOF
\$(file >test.txt,Success )

.PHONY: all
all:
<TAB>@cat test.txt

EOF
$GNUMAKE -C $TESTGMAKEFILEFUNC 2>/dev/null 1>&2
if test -f $TESTGMAKEFILEFUNC/test.txt; then
    HAVE_GNUMAKE_FILE_FUNC=TRUE
    AC_MSG_RESULT([yes])
else
    AC_MSG_RESULT([no])
fi
rm -rf $TESTGMAKEFILEFUNC
AC_SUBST(HAVE_GNUMAKE_FILE_FUNC)
AC_SUBST(GNUMAKE_WIN_NATIVE)

_make_ver_check=`$GNUMAKE --version | grep "Built for Windows"`
STALE_MAKE=
if test "$_make_ver_check" = ""; then
   STALE_MAKE=TRUE
fi

if test "$GNUMAKE_WIN_NATIVE" = "TRUE" ; then
    SHELL_BASH=$(cygpath -m $(which bash))
else
    SHELL_BASH=`which bash`
fi
AC_SUBST(SHELL_BASH)

HAVE_LD_HASH_STYLE=FALSE
WITH_LINKER_HASH_STYLE=
AC_MSG_CHECKING( for --hash-style gcc linker support )
if test "$GCC" = "yes"; then
    if test -z "$with_linker_hash_style" -o "$with_linker_hash_style" = "yes"; then
        hash_styles="gnu sysv"
    elif test "$with_linker_hash_style" = "no"; then
        hash_styles=
    else
        hash_styles="$with_linker_hash_style"
    fi

    for hash_style in $hash_styles; do
        test "$HAVE_LD_HASH_STYLE" = "TRUE" && continue
        hash_style_ldflags_save=$LDFLAGS
        LDFLAGS="$LDFLAGS -Wl,--hash-style=$hash_style"

        AC_RUN_IFELSE([AC_LANG_PROGRAM(
            [
#include <stdio.h>
            ],[
printf ("");
            ])],
            [
                  HAVE_LD_HASH_STYLE=TRUE
                  WITH_LINKER_HASH_STYLE=$hash_style
            ],
            [HAVE_LD_HASH_STYLE=FALSE],
            [HAVE_LD_HASH_STYLE=FALSE])
        LDFLAGS=$hash_style_ldflags_save
    done

    if test "$HAVE_LD_HASH_STYLE" = "TRUE"; then
        AC_MSG_RESULT( $WITH_LINKER_HASH_STYLE )
    else
        AC_MSG_RESULT( no )
    fi
    LDFLAGS=$hash_style_ldflags_save
else
    AC_MSG_RESULT( no )
fi
AC_SUBST(HAVE_LD_HASH_STYLE)
AC_SUBST(WITH_LINKER_HASH_STYLE)

dnl ===================================================================
dnl Check whether there's a Perl version available.
dnl ===================================================================
if test -z "$with_perl_home"; then
    AC_PATH_PROG(PERL, perl)
else
    test "$build_os" = "cygwin" && with_perl_home=`cygpath -u "$with_perl_home"`
    _perl_path="$with_perl_home/bin/perl"
    if test -x "$_perl_path"; then
        PERL=$_perl_path
    else
        AC_MSG_ERROR([$_perl_path not found])
    fi
fi

dnl ===================================================================
dnl Testing for Perl version 5 or greater.
dnl $] is the Perl version variable, it is returned as an integer
dnl ===================================================================
if test "$PERL"; then
    AC_MSG_CHECKING([the Perl version])
    ${PERL} -e "exit($]);"
    _perl_version=$?
    if test "$_perl_version" -lt 5; then
        AC_MSG_ERROR([found Perl version "$_perl_version", use version 5 of Perl])
    fi
    AC_MSG_RESULT([checked (perl $_perl_version)])
else
    AC_MSG_ERROR([Perl not found, install version 5 of Perl])
fi

dnl ===================================================================
dnl Testing for required Perl modules
dnl ===================================================================

AC_MSG_CHECKING([for required Perl modules])
if `$PERL -e 'use Archive::Zip; use Cwd; use Digest::MD5'>/dev/null 2>&1`; then
    AC_MSG_RESULT([all modules found])
else
    AC_MSG_RESULT([failed to find some modules])
    # Find out which modules are missing.
    missing_perl_modules=
    if ! `$PERL -e 'use Archive::Zip;'>/dev/null 2>&1`; then
        missing_perl_modules=Archive::Zip
    fi
    if ! `$PERL -e 'use Cwd;'>/dev/null 2>&1`; then
        missing_perl_modules="$missing_perl_modules Cwd"
    fi
    if ! `$PERL -e 'use Digest::MD5;'>/dev/null 2>&1`; then
        missing_perl_modules="$missing_perl_modules Digest::MD5"
    fi
       AC_MSG_ERROR([
    The missing Perl modules are: $missing_perl_modules
    Install them as superuser/administrator with "cpan -i $missing_perl_modules"])
fi

dnl ===================================================================
dnl Check for pkg-config
dnl ===================================================================
if test "$_os" != "WINNT" -o "$WITH_MINGW" = "yes"; then
    PKG_PROG_PKG_CONFIG
fi

if test "$_os" != "WINNT" -o "$WITH_MINGW" = "yes"; then

    # If you use CC=/path/to/compiler/foo-gcc or even CC="ccache
    # /path/to/compiler/foo-gcc" you need to set the AR etc env vars
    # explicitly. Or put /path/to/compiler in PATH yourself.

    AC_CHECK_TOOL(AR,ar)
    AC_CHECK_TOOL(NM,nm)
    AC_CHECK_TOOL(OBJDUMP,objdump)
    AC_CHECK_TOOL(RANLIB,ranlib)
    AC_CHECK_TOOL(STRIP,strip)
    if test "$_os" = "WINNT"; then
        AC_CHECK_TOOL(DLLTOOL,dlltool)
        AC_CHECK_TOOL(WINDRES,windres)
    fi
fi
AC_SUBST(AR)
AC_SUBST(DLLTOOL)
AC_SUBST(NM)
AC_SUBST(OBJDUMP)
AC_SUBST(PKG_CONFIG)
AC_SUBST(RANLIB)
AC_SUBST(STRIP)
AC_SUBST(WINDRES)

dnl ===================================================================
dnl pkg-config checks on Mac OS X
dnl ===================================================================

if test $_os = Darwin; then
    AC_MSG_CHECKING([for bogus pkg-config])
    if test -n "$PKG_CONFIG"; then
        if test "$PKG_CONFIG" = /usr/bin/pkg-config && ls -l /usr/bin/pkg-config | $GREP -q Mono.framework; then
            AC_MSG_ERROR([yes, from Mono. This *will* break the build. Please remove or hide $PKG_CONFIG])
        else
            if test "$enable_bogus_pkg_config" = "yes"; then
                AC_MSG_RESULT([yes, user-approved from unknown origin.])
            else
                AC_MSG_ERROR([yes, from unknown origin. This *will* break the build. Please remove or hide $PKG_CONFIG])
            fi
        fi
    else
        AC_MSG_RESULT([no, good])
    fi
fi

find_csc()
{
    # Return value: $csctest

    unset csctest

    reg_get_value_32 "HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/NET Framework Setup/NDP/v4/Client/InstallPath"
    if test -n "$regvalue"; then
        csctest=$regvalue
        return
    fi
}

find_al()
{
    # Return value: $altest

    unset altest

    for x in `ls /proc/registry32/HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Microsoft\ SDKs/Windows`; do
        reg_get_value_32 "HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Microsoft SDKs/Windows/$x/WinSDK-NetFx40Tools/InstallationFolder"
        if test -n "$regvalue" -a \( -f "$regvalue/al.exe" -o -f "$regvalue/bin/al.exe" \); then
            altest=$regvalue
            return
        fi
    done
}

find_dotnetsdk()
{
    # Return value: $frametest (that's a silly name...)

    unset frametest

    for ver in 1.1 2.0; do
        reg_get_value_32 HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/.NETFramework/sdkInstallRootv$ver
        if test -n "$regvalue"; then
            frametest=$regvalue
            return
        fi
    done
}

find_winsdk_version()
{
    # Args: $1 : SDK version as in "6.0A", "7.0" etc
    # Return values: $winsdktest, $winsdklibsubdir

    unset winsdktest winsdklibsubdir

    # Why we look for them in this particular order I don't know. But OTOH I
    case "$1" in
    6.0*|7.*)
        reg_get_value_32 "HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Microsoft SDKs/Windows/v${1}/InstallationFolder"
        if test -n "$regvalue"; then
            winsdktest=$regvalue
            winsdklibsubdir=.
            return
        fi
        ;;
    8.1)
        reg_get_value_32 "HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows Kits/Installed Roots/KitsRoot81"
        if test -n "$regvalue"; then
            winsdktest=$regvalue
            winsdklibsubdir=winv6.3
            return
        fi
        ;;
    8.*)
        reg_get_value_32 "HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows Kits/Installed Roots/KitsRoot"
        if test -n "$regvalue"; then
            winsdktest=$regvalue
            winsdklibsubdir=win8
            return
        fi
        ;;
    esac
}

find_winsdk()
{
    # Args: $1 (optional) : list of acceptable SDK versions
    # Return value: $winsdktest

    unset winsdktest

    if test -n "$1"; then
        sdkversions=$1
    else
        sdkversions="$WINDOWS_SDK_ACCEPTABLE_VERSIONS"
    fi

    for ver in $sdkversions; do
        find_winsdk_version $ver
        if test -n "$winsdktest"; then
            return
        fi
    done
}

find_msms()
{
    for ver in 10.0 11.0; do
        reg_get_value_32 HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/VisualStudio/$ver/Setup/VS/MSMDir
        if test -n "$regvalue"; then
            if test -e "$regvalue/Microsoft_VC${VCVER}_CRT_x86.msm"; then
                msmdir=$regvalue
                break
            fi
        fi
    done
    if test -z "$msmdir"; then
        AC_MSG_NOTICE([no registry entry for Merge Module directory - trying "${COMMONPROGRAMFILES}/Merge Modules/"])
        msmdir="${COMMONPROGRAMFILES}/Merge Modules/"
    fi
    msmdir=`cygpath -m "$msmdir"`
    if test -z "$msmdir"; then
        if test "$ENABLE_RELEASE_BUILD" = "TRUE" ; then
            AC_MSG_ERROR([Merge modules not found in $msmdir])
        else
            AC_MSG_WARN([Merge modules not found in $msmdir])
            msmdir=""
        fi
    fi
}

find_msvc_x64_dlls()
{
    msvcdllpath="$VC_PRODUCT_DIR/redist/x64/Microsoft.VC${VCVER}.CRT"
    msvcdlls="msvcp${VCVER}.dll msvcr${VCVER}.dll"
    for dll in $msvcdlls; do
        if ! test -f "$msvcdllpath/$dll"; then
            AC_MSG_ERROR([can not find $dll in $msvcdllpath])
        fi
    done
}

if test "$build_os" = "cygwin"; then
    dnl Check midl.exe
    AC_MSG_CHECKING([for midl.exe])

    find_winsdk
    if test -f "$winsdktest/Bin/midl.exe"; then
        MIDL_PATH="$winsdktest/Bin"
    elif test -f "$winsdktest/Bin/$WINDOWS_SDK_ARCH/midl.exe"; then
        MIDL_PATH="$winsdktest/Bin/$WINDOWS_SDK_ARCH"
    fi
    if test ! -f "$MIDL_PATH/midl.exe"; then
        AC_MSG_ERROR([midl.exe not found in $winsdktest/Bin/$WINDOWS_SDK_ARCH, Windows SDK installation broken?])
    else
        AC_MSG_RESULT([$MIDL_PATH/midl.exe])
    fi

    # Convert to posix path with 8.3 filename restrictions ( No spaces )
    MIDL_PATH=`win_short_path_for_make "$MIDL_PATH"`

    dnl Check csc.exe
    AC_MSG_CHECKING([for csc.exe])
    find_csc
    if test -f "$csctest/csc.exe"; then
        CSC_PATH="$csctest"
    fi
    if test ! -f "$CSC_PATH/csc.exe"; then
        AC_MSG_ERROR([csc.exe not found as $CSC_PATH/csc.exe])
    else
        AC_MSG_RESULT([$CSC_PATH/csc.exe])
    fi

    CSC_PATH=`win_short_path_for_make "$CSC_PATH"`

    dnl Check al.exe
    AC_MSG_CHECKING([for al.exe])
    find_winsdk
    if test -f "$winsdktest/Bin/al.exe"; then
        AL_PATH="$winsdktest/Bin"
    elif test -f "$winsdktest/Bin/$WINDOWS_SDK_ARCH/al.exe"; then
        AL_PATH="$winsdktest/Bin/$WINDOWS_SDK_ARCH"
    fi

    if test -z "$AL_PATH"; then
        find_al
        if test -f "$altest/bin/al.exe"; then
            AL_PATH="$altest/bin"
        elif test -f "$altest/al.exe"; then
            AL_PATH="$altest"
        fi
    fi
    if test ! -f "$AL_PATH/al.exe"; then
        AC_MSG_ERROR([al.exe not found as $AL_PATH/al.exe])
    else
        AC_MSG_RESULT([$AL_PATH/al.exe])
    fi

    AL_PATH=`win_short_path_for_make "$AL_PATH"`

    dnl Check mscoree.lib / .NET Framework dir
    AC_MSG_CHECKING(.NET Framework)
    find_dotnetsdk
    if test -f "$frametest/lib/mscoree.lib"; then
        DOTNET_FRAMEWORK_HOME="$frametest"
    else
        find_winsdk
        if test -f "$winsdktest/lib/mscoree.lib" -o -f "$winsdktest/lib/$winsdklibsubdir/um/$WINDOWS_SDK_ARCH/mscoree.lib"; then
            DOTNET_FRAMEWORK_HOME="$winsdktest"
        fi
    fi

    if test ! -f "$DOTNET_FRAMEWORK_HOME/lib/mscoree.lib" -a ! -f "$DOTNET_FRAMEWORK_HOME/lib/$winsdklibsubdir/um/$WINDOWS_SDK_ARCH/mscoree.lib"; then
        AC_MSG_ERROR([mscoree.lib not found])
    fi
    AC_MSG_RESULT(found)

    PathFormat "$MIDL_PATH"
    MIDL_PATH="$formatted_path"

    PathFormat "$AL_PATH"
    AL_PATH="$formatted_path"

    PathFormat "$DOTNET_FRAMEWORK_HOME"
    DOTNET_FRAMEWORK_HOME="$formatted_path"

    PathFormat "$CSC_PATH"
    CSC_PATH="$formatted_path"
fi

dnl ===================================================================
dnl Check if stdc headers are available excluding MSVC.
dnl ===================================================================
if test "$_os" != "WINNT" -o "$WITH_MINGW" = "yes"; then
    AC_HEADER_STDC
fi

dnl ===================================================================
dnl Testing for C++ compiler and version...
dnl ===================================================================

if test "$_os" != "WINNT" -o "$WITH_MINGW" = "yes"; then
    # AC_PROG_CXX sets CXXFLAGS to -g -O2 if not set, avoid that
    save_CXXFLAGS=$CXXFLAGS
    AC_PROG_CXX
    CXXFLAGS=$save_CXXFLAGS
else
    if test -n "$CC" -a -z "$CXX"; then
        CXX="$CC"
    fi
fi

dnl check for GNU C++ compiler version
if test "$GXX" = "yes"; then
    AC_MSG_CHECKING([the GNU C++ compiler version])

    _gpp_version=`$CXX -dumpversion`
    _gpp_majmin=`echo $_gpp_version | $AWK -F. '{ print \$1*100+\$2 }'`

    if test "$_gpp_majmin" -lt "401"; then
        AC_MSG_ERROR([You need to use GNU C++ compiler version >= 4.1 to build LibreOffice, you have $_gpp_version.])
    else
        AC_MSG_RESULT([checked (g++ $_gpp_version)])
    fi

    dnl see https://code.google.com/p/android/issues/detail?id=41770
    if test "$_gpp_majmin" -ge "401"; then
        glibcxx_threads=no
        AC_LANG_PUSH([C++])
        AC_REQUIRE_CPP
        AC_MSG_CHECKING([whether $CXX is broken with boost.thread])
        AC_PREPROC_IFELSE([AC_LANG_PROGRAM([[
            #include <bits/c++config.h>]],[[
            #if !defined(_GLIBCXX_HAVE_GTHR_DEFAULT) \
            && !defined(_GLIBCXX__PTHREADS) \
            && !defined(_GLIBCXX_HAS_GTHREADS)
            choke me
            #endif
        ]])],[AC_MSG_RESULT([yes])
        glibcxx_threads=yes],[AC_MSG_RESULT([no])])
        AC_LANG_POP([C++])
        if test $glibcxx_threads = yes; then
            BOOST_CXXFLAGS="-D_GLIBCXX_HAS_GTHREADS"
        fi
     fi
fi
AC_SUBST(BOOST_CXXFLAGS)

#
# prefx CXX with ccache if needed
#
if test "$_os" != "WINNT" -o "$WITH_MINGW" = "yes"; then
    if test "$CCACHE" != ""; then
        AC_MSG_CHECKING([whether $CXX is already ccached])
        AC_LANG_PUSH([C++])
        save_CXXFLAGS=$CXXFLAGS
        CXXFLAGS="$CXXFLAGS --ccache-skip -O2"
        dnl an empty program will do, we're checking the compiler flags
        AC_COMPILE_IFELSE([AC_LANG_PROGRAM([],[])],
                          [use_ccache=yes], [use_ccache=no])
        if test $use_ccache = yes; then
            AC_MSG_RESULT([yes])
        else
            CXX="$CCACHE $CXX"
            AC_MSG_RESULT([no])
        fi
        CXXFLAGS=$save_CXXFLAGS
        AC_LANG_POP([C++])
    fi
fi

dnl ===================================================================
dnl Find pre-processors.(should do that _after_ messing with CC/CXX)
dnl ===================================================================

if test "$_os" != "WINNT" -o "$WITH_MINGW" = "yes"; then
    AC_PROG_CXXCPP

    dnl Check whether there's a C pre-processor.
    AC_PROG_CPP
fi


dnl ===================================================================
dnl Find integral type sizes and alignments
dnl ===================================================================

if test "$_os" != "WINNT" -o "$WITH_MINGW" = "yes"; then

    AC_CHECK_SIZEOF(long)
    AC_CHECK_SIZEOF(short)
    AC_CHECK_SIZEOF(int)
    AC_CHECK_SIZEOF(long long)
    AC_CHECK_SIZEOF(double)
    AC_CHECK_SIZEOF(void*)

    SAL_TYPES_SIZEOFSHORT=$ac_cv_sizeof_short
    SAL_TYPES_SIZEOFINT=$ac_cv_sizeof_int
    SAL_TYPES_SIZEOFLONG=$ac_cv_sizeof_long
    SAL_TYPES_SIZEOFLONGLONG=$ac_cv_sizeof_long_long
    SAL_TYPES_SIZEOFPOINTER=$ac_cv_sizeof_voidp

    dnl Allow build without AC_CHECK_ALIGNOF, grrr
    m4_pattern_allow([AC_CHECK_ALIGNOF])
    m4_ifdef([AC_CHECK_ALIGNOF],
        [
            AC_CHECK_ALIGNOF(short,[#include <stddef.h>])
            AC_CHECK_ALIGNOF(int,[#include <stddef.h>])
            AC_CHECK_ALIGNOF(long,[#include <stddef.h>])
            AC_CHECK_ALIGNOF(double,[#include <stddef.h>])
        ],
        [
            case "$_os-$host_cpu" in
            Linux-i686)
                test -z "$ac_cv_alignof_short" && ac_cv_alignof_short=2
                test -z "$ac_cv_alignof_int" && ac_cv_alignof_int=4
                test -z "$ac_cv_alignof_long" && ac_cv_alignof_long=4
                test -z "$ac_cv_alignof_double" && ac_cv_alignof_double=4
                ;;
            Linux-x86_64)
                test -z "$ac_cv_alignof_short" && ac_cv_alignof_short=2
                test -z "$ac_cv_alignof_int" && ac_cv_alignof_int=4
                test -z "$ac_cv_alignof_long" && ac_cv_alignof_long=8
                test -z "$ac_cv_alignof_double" && ac_cv_alignof_double=8
                ;;
            *)
                if test -z "$ac_cv_alignof_short" -o \
                        -z "$ac_cv_alignof_int" -o \
                        -z "$ac_cv_alignof_long" -o \
                        -z "$ac_cv_alignof_double"; then
                   AC_MSG_ERROR([Your Autoconf doesn't have [AC_][CHECK_ALIGNOF]. You need to set the environment variables ac_cv_alignof_short, ac_cv_alignof_int, ac_cv_alignof_long and ac_cv_alignof_double.])
                fi
                ;;
            esac
        ])

    SAL_TYPES_ALIGNMENT2=$ac_cv_alignof_short
    SAL_TYPES_ALIGNMENT4=$ac_cv_alignof_int
    if test $ac_cv_sizeof_long -eq 8; then
        SAL_TYPES_ALIGNMENT8=$ac_cv_alignof_long
    elif test $ac_cv_sizeof_double -eq 8; then
        SAL_TYPES_ALIGNMENT8=$ac_cv_alignof_double
    else
        AC_MSG_ERROR([Cannot find alignment of 8 byte types.])
    fi

    dnl Check for large file support
    AC_SYS_LARGEFILE
    if test -n "$ac_cv_sys_file_offset_bits" -a "$ac_cv_sys_file_offset_bits" != "no"; then
        LFS_CFLAGS="-D_FILE_OFFSET_BITS=$ac_cv_sys_file_offset_bits"
    fi
    if test -n "$ac_cv_sys_large_files" -a "$ac_cv_sys_large_files" != "no"; then
        LFS_CFLAGS="$LFS_CFLAGS -D_LARGE_FILES"
    fi
else
    # Hardcode for MSVC
    SAL_TYPES_SIZEOFSHORT=2
    SAL_TYPES_SIZEOFINT=4
    SAL_TYPES_SIZEOFLONG=4
    SAL_TYPES_SIZEOFLONGLONG=8
    if test "$BITNESS_OVERRIDE" = ""; then
        SAL_TYPES_SIZEOFPOINTER=4
    else
        SAL_TYPES_SIZEOFPOINTER=8
    fi
    SAL_TYPES_ALIGNMENT2=2
    SAL_TYPES_ALIGNMENT4=4
    SAL_TYPES_ALIGNMENT8=8
    LFS_CFLAGS=''
fi
AC_SUBST(LFS_CFLAGS)

AC_DEFINE_UNQUOTED(SAL_TYPES_SIZEOFSHORT,$SAL_TYPES_SIZEOFSHORT)
AC_DEFINE_UNQUOTED(SAL_TYPES_SIZEOFINT,$SAL_TYPES_SIZEOFINT)
AC_DEFINE_UNQUOTED(SAL_TYPES_SIZEOFLONG,$SAL_TYPES_SIZEOFLONG)
AC_DEFINE_UNQUOTED(SAL_TYPES_SIZEOFLONGLONG,$SAL_TYPES_SIZEOFLONGLONG)
AC_DEFINE_UNQUOTED(SAL_TYPES_SIZEOFPOINTER,$SAL_TYPES_SIZEOFPOINTER)
AC_DEFINE_UNQUOTED(SAL_TYPES_ALIGNMENT2,$SAL_TYPES_ALIGNMENT2)
AC_DEFINE_UNQUOTED(SAL_TYPES_ALIGNMENT4,$SAL_TYPES_ALIGNMENT4)
AC_DEFINE_UNQUOTED(SAL_TYPES_ALIGNMENT8,$SAL_TYPES_ALIGNMENT8)

dnl ===================================================================
dnl Check whether to enable runtime optimizations
dnl ===================================================================
ENABLE_RUNTIME_OPTIMIZATIONS=
AC_MSG_CHECKING([whether to enable runtime optimizations])
if test -z "$enable_runtime_optimizations"; then
    for i in $CC; do
        case $i in
        -fsanitize=*)
            enable_runtime_optimizations=no
            break
            ;;
        esac
    done
fi
if test "$enable_runtime_optimizations" != no; then
    ENABLE_RUNTIME_OPTIMIZATIONS=TRUE
    AC_DEFINE(ENABLE_RUNTIME_OPTIMIZATIONS)
    AC_MSG_RESULT([yes])
else
    AC_MSG_RESULT([no])
fi
AC_SUBST([ENABLE_RUNTIME_OPTIMIZATIONS])

dnl ===================================================================
dnl Check if valgrind headers are available
dnl ===================================================================
ENABLE_VALGRIND=
if test "$cross_compiling" != yes; then
    prev_cppflags=$CPPFLAGS
    # Is VALGRIND_CFLAGS something one is supposed to have in the environment,
    # or where does it come from?
    CPPFLAGS="$CPPFLAGS $VALGRIND_CFLAGS"
    AC_CHECK_HEADER([valgrind/valgrind.h],
        [ENABLE_VALGRIND=TRUE])
    CPPFLAGS=$prev_cppflags
fi
AC_SUBST([ENABLE_VALGRIND])
if test -z "$ENABLE_VALGRIND"; then
    VALGRIND_CFLAGS=
fi
AC_SUBST([VALGRIND_CFLAGS])


dnl ===================================================================
dnl Check if SDT probes (for systemtap, gdb, dtrace) are available
dnl ===================================================================

# We need at least the sys/sdt.h include header.
AC_CHECK_HEADER([sys/sdt.h], [SDT_H_FOUND='TRUE'], [SDT_H_FOUND='FALSE'])
if test "$SDT_H_FOUND" = "TRUE"; then
    # Found sys/sdt.h header, now make sure the c++ compiler works.
    # Old g++ versions had problems with probes in constructors/destructors.
    AC_MSG_CHECKING([working sys/sdt.h and c++ support])
    AC_LANG_PUSH([C++])
    AC_LINK_IFELSE([AC_LANG_PROGRAM([[
    #include <sys/sdt.h>
    class ProbeClass
    {
    private:
      int& ref;
      const char *name;

    public:
      ProbeClass(int& v, const char *n) : ref(v), name(n)
      {
        DTRACE_PROBE2(_test_, cons, name, ref);
      }

      void method(int min)
      {
        DTRACE_PROBE3(_test_, meth, name, ref, min);
        ref -= min;
      }

      ~ProbeClass()
      {
        DTRACE_PROBE2(_test_, dest, name, ref);
      }
    };
    ]],[[
    int i = 64;
    DTRACE_PROBE1(_test_, call, i);
    ProbeClass inst = ProbeClass(i, "call");
    inst.method(24);
    ]])], [AC_MSG_RESULT([yes]); AC_DEFINE([USE_SDT_PROBES])],
          [AC_MSG_RESULT([no, sdt.h or c++ compiler too old])])
    AC_LANG_POP([C++])
fi

dnl ===================================================================
dnl Set the MinGW sys-root
dnl ===================================================================
if test "$WITH_MINGW" = "yes"; then
    AC_MSG_CHECKING([for MinGW sysroot])
    sysroot=`$CC -print-sysroot`
    AS_IF([test -d "$sysroot"/mingw],
          [MINGW_SYSROOT="$sysroot"/mingw
           AC_MSG_RESULT([$MINGW_SYSROOT])],
          [AC_MSG_RESULT([not found])
           AC_MSG_ERROR([cannot determine MinGW sysroot])])
fi
AC_SUBST([MINGW_DLLS])
AC_SUBST([MINGW_SYSROOT])

dnl ===================================================================
dnl Set the MinGW include directories
dnl ===================================================================
if test "$WITH_MINGW" = "yes"; then
    AC_MSG_CHECKING([for MinGW include path])
    cat >conftest.$ac_ext <<_ACEOF
#include <stddef.h>
#include <bits/c++config.h>
_ACEOF
    _mingw_lib_include_path=`$CXX -E -xc++ conftest.$ac_ext | $SED -n -e '/.*1*"\(.*\)\/stddef.h".*/s//\1/p' -e '/.*1*"\(.*\)\/bits\/c++config.h".*/s//\1/p' | sort -u | xargs echo`
    rm conftest.$ac_ext
    if test -z "$_mingw_lib_include_path"; then
        _mingw_lib_include_path="NO_LIB_INCLUDE"
        AC_MSG_RESULT([no MinGW include path])
    else
        AC_MSG_RESULT([$_mingw_lib_include_path])
    fi
    MINGW_LIB_INCLUDE_PATH="$_mingw_lib_include_path"
    AC_SUBST(MINGW_LIB_INCLUDE_PATH)

    AC_LANG_PUSH([C++])

    AC_MSG_CHECKING([for dynamic libgcc])
    AC_LINK_IFELSE([AC_LANG_PROGRAM([[
#include <iostream>
using namespace std;
]], [[ try { throw 42; } catch (int e) { cout << "Yep, " << e << endl; } ]])],[
            MINGW_GCCDLL=`$OBJDUMP -p conftest.exe | grep 'DLL Name: libgcc' | $SED -e 's@.*DLL Name: @@'`
            if test -n "$MINGW_GCCDLL"; then
                MINGW_SHARED_GCCLIB=TRUE
                AC_MSG_RESULT([ $MINGW_GCCDLL])
            else
                MINGW_SHARED_GCCLIB=
                AC_MSG_RESULT([no])
            fi
       ],[ AC_MSG_RESULT([no])

    ])

    AC_MSG_CHECKING([for dynamic libstdc++])
    AC_LINK_IFELSE([AC_LANG_PROGRAM([[
#include <iostream>
using namespace std;
]], [[ cout << "Hello there." << endl; ]])],[
            MINGW_GXXDLL=`$OBJDUMP -p conftest.exe | grep 'DLL Name: libstdc++' | $SED -e 's@.*DLL Name: @@'`
            if test -n "$MINGW_GXXDLL"; then
                mingw_gxxdll_root=${MINGW_GXXDLL%.dll}
                mingw_gxxdll_root=${mingw_gxxdll_root#lib}
                if test "$CROSS_COMPILING" = "TRUE"; then
                    dnl m4 escaping!
                    mingw_gxxdll_root=${mingw_gxxdll_root%-[[0-9]]}
                fi
                MINGW_SHARED_LIBSTDCPP=-l$mingw_gxxdll_root
                MINGW_SHARED_GXXLIB=TRUE
                AC_MSG_RESULT([$MINGW_GXXDLL])
            else
                MINGW_SHARED_GXXLIB=
                AC_MSG_RESULT([no])
            fi
       ],[ AC_MSG_RESULT([no])

    ])

    AC_LANG_POP([C++])

    AC_SUBST(MINGW_SHARED_GCCLIB)
    AC_SUBST(MINGW_SHARED_GXXLIB)
    AC_SUBST(MINGW_SHARED_LIBSTDCPP)
    AC_SUBST(MINGW_GCCDLL)
    AC_SUBST(MINGW_GXXDLL)
fi

if test "$_os" = "WINNT" -a "$WITH_MINGW" != "yes"; then
    AC_DEFINE(HAVE_FMERGENEUTRALITEMS)
fi

if test "$WITH_MINGW" = "yes"; then
    AC_MSG_CHECKING([for fMergeNeutralItems in SCRIPT_CONTROL])
    AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
    [
    #include <usp10.h>
    ],
    [
    SCRIPT_CONTROL c;
    c.fMergeNeutralItems = 1;
    ])],
    [
    AC_MSG_RESULT(yes)
    AC_DEFINE(HAVE_FMERGENEUTRALITEMS)
    ],
    [AC_MSG_RESULT(no)])
fi

dnl *************************************************************
dnl Testing for exception handling - dwarf2 or sjlj exceptions...
dnl *************************************************************
if test "$WITH_MINGW" = "yes"; then
    AC_MSG_CHECKING([exception type])
    AC_LANG_PUSH([C++])
    AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <iostream>

        extern "C" void _Unwind_SjLj_RaiseException(void) __attribute__ ((__noreturn__));

        ]], [[_Unwind_SjLj_RaiseException() ]])],[exceptions_type="sjlj"],[exceptions_type="dwarf2"
    ])
    AC_MSG_RESULT($exceptions_type)
    AC_LANG_POP([C++])
fi

EXCEPTIONS="$exceptions_type"
AC_SUBST(EXCEPTIONS)

dnl ===================================================================
dnl thread-safe statics
dnl ===================================================================
AC_MSG_CHECKING([whether $CXX supports thread safe statics])
unset HAVE_THREADSAFE_STATICS
if test "$GCC" = "yes"; then
    dnl -fthreadsafe-statics is available since GCC 4, so always available for
    dnl us.  However, some C++ runtimes use a single lock for all static
    dnl variables, which can cause deadlock in multi-threaded applications.
    dnl This is not easily tested here; for POSIX-based systems, if executing
    dnl the following C++ program does not terminate then the tool chain
    dnl apparently has this problem:
    dnl
    dnl   #include <pthread.h>
    dnl   int h() { return 0; }
    dnl   void * g(void * unused) {
    dnl     static int n = h();
    dnl     return &n;
    dnl   }
    dnl   int f() {
    dnl     pthread_t t;
    dnl     pthread_create(&t, 0, g, 0);
    dnl     pthread_join(t, 0);
    dnl     return 0;
    dnl   }
    dnl   int main() {
    dnl     static int n = f();
    dnl     return n;
    dnl   }
    dnl
    dnl Mac OS X up to at least 10.7.1 is known to have this problem, as is at
    dnl least one instance of GCC 4.2.4 (used on a "Linux ooobuild1.osuosl.org
    dnl 2.6.9-101.plus.c4smp #1 SMP Thu Jul 21 19:08:15 EDT 2011 i686 i686 i386
    dnl GNU/Linux" machine); see the definition of __cxa_guard_acquire in GCC's
    dnl libstdc++-v3/libsupc++/guard.cc for what #ifdefs actually make a
    dnl difference there.  Conservative advice from Jakub Jelinek is to assume
    dnl it working in GCC >= 4.3, so conservative way to check here is to use
    dnl GCC_VERSION for GCC but resort to __GLIBCXX__ corresponding to libstdc++
    dnl shipped with GCC 4.3.0 (cf. <http://gcc.gnu.org/onlinedocs/libstdc++/
    dnl manual/abi.html#abi.versioning.history>; 4.3.0 is 20080306, 4.2.4 is
    dnl 20080519, 4.3.1 is 20080606) for Clang (for which GCC_VERSION is notoriously
    dnl "too old"):
    if test "$_os" = Darwin; then
        dnl On Mac OS X, both /usr/lib/libstdc++.6.dylib and
        dnl /usr/lib/libc++1.dylib delegate to /usr/lib/libc++abi.dylib for the
        dnl __cxa_guard_* functionality; up to 10.8, libc++abi.dylib is
        dnl apparently provided by a libcppabi project that has broken semantics
        dnl (cf. <http://www.opensource.apple.com/source/libcppabi/libcppabi-26/
        dnl src/cxa_guard.cxx>), while in 10.9 (verified at least on 10.9.4) it
        dnl is apparently provided by LLVM's libcxxabi project, which has good
        dnl semantics (cf. <http://llvm.org/svn/llvm-project/libcxxabi/trunk/
        dnl src/cxa_guard.cpp>):
        if test "$MAC_OS_X_VERSION_MIN_REQUIRED" -ge 1090; then
            HAVE_THREADSAFE_STATICS=TRUE
        fi
    elif test "$_os" != Android; then
        if test "$COM_GCC_IS_CLANG" = TRUE; then
            AC_LANG_PUSH([C++])
            AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
#include <list>
#if !defined __GLIBCXX__ ||(__GLIBCXX__ < 20080606 && __GLIBCXX__ != 20080306)
#error
#endif
                ]])],[HAVE_THREADSAFE_STATICS=TRUE],[])
            AC_LANG_POP([C++])
        else # known to work in GCC since version 4.3
            HAVE_THREADSAFE_STATICS=TRUE
        fi
    fi
    if test "$HAVE_THREADSAFE_STATICS" = TRUE; then
        AC_DEFINE(HAVE_THREADSAFE_STATICS)
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([broken (i.e., no)])
    fi
else
    AC_MSG_RESULT([unknown (assuming no)])
fi
AC_SUBST(HAVE_THREADSAFE_STATICS)

dnl ===================================================================
dnl visibility and other gcc features
dnl ===================================================================
if test "$GCC" = "yes"; then
    AC_MSG_CHECKING([whether $CC supports -fvisibility=hidden])
    save_CFLAGS=$CFLAGS
    CFLAGS="$CFLAGS -Werror -fvisibility=hidden"
    AC_LINK_IFELSE([AC_LANG_PROGRAM([[]], [[ return 0; ]])],[ HAVE_GCC_VISIBILITY_FEATURE=TRUE ],[])
    CFLAGS=$save_CFLAGS

    if test "$HAVE_GCC_VISIBILITY_FEATURE" = "TRUE"; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi

    AC_MSG_CHECKING([whether $CC supports -mno-avx])
    save_CFLAGS=$CFLAGS
    CFLAGS="$CFLAGS -Werror -mno-avx"
    AC_LINK_IFELSE([AC_LANG_PROGRAM([[]], [[ return 0; ]])],[ HAVE_GCC_AVX=TRUE ],[])
    CFLAGS=$save_CFLAGS
    if test "$HAVE_GCC_AVX" = "TRUE"; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi

    AC_MSG_CHECKING([whether $CC supports atomic functions])
    AC_LINK_IFELSE([AC_LANG_PROGRAM([[]], [[
    int v = 0;
    if (__sync_add_and_fetch(&v, 1) != 1 ||
        __sync_sub_and_fetch(&v, 1) != 0)
        return 1;
    __sync_synchronize();
    if (__sync_val_compare_and_swap(&v, 0, 1) != 0 ||
        v != 1)
        return 1;
    return 0;
]])],[HAVE_GCC_BUILTIN_ATOMIC=TRUE],[])
    if test "$HAVE_GCC_BUILTIN_ATOMIC" = "TRUE"; then
        AC_MSG_RESULT([yes])
        AC_DEFINE(HAVE_GCC_BUILTIN_ATOMIC)
    else
        AC_MSG_RESULT([no])
    fi

    AC_MSG_CHECKING([whether $CC supports __attribute__((deprecated(message)))])
    save_CFLAGS=$CFLAGS
    CFLAGS="$CFLAGS -Werror"
    AC_COMPILE_IFELSE([AC_LANG_SOURCE([
            __attribute__((deprecated("test"))) void f();
        ])], [
            AC_DEFINE([HAVE_GCC_DEPRECATED_MESSAGE],[1])
            AC_MSG_RESULT([yes])
        ], [AC_MSG_RESULT([no])])
    CFLAGS=$save_CFLAGS

    AC_MSG_CHECKING([whether $CXX declares __cxa_allocate_exception in cxxabi.h])
    AC_LANG_PUSH([C++])
    AC_COMPILE_IFELSE([AC_LANG_SOURCE([
            #include <cxxabi.h>
            void * f() { return __cxxabiv1::__cxa_allocate_exception(0); }
        ])], [
            AC_DEFINE([HAVE_GCC_CXXABI_H_CXA_ALLOCATE_EXCEPTION],[1])
            AC_MSG_RESULT([yes])
        ], [AC_MSG_RESULT([no])])
    AC_LANG_POP([C++])

    AC_MSG_CHECKING([whether $CXX declares __cxa_get_globals in cxxabi.h])
    AC_LANG_PUSH([C++])
    AC_COMPILE_IFELSE([AC_LANG_SOURCE([
            #include <cxxabi.h>
            void * f() { return __cxxabiv1::__cxa_get_globals(); }
        ])], [
            AC_DEFINE([HAVE_GCC_CXXABI_H_CXA_GET_GLOBALS],[1])
            AC_MSG_RESULT([yes])
        ], [AC_MSG_RESULT([no])])
    AC_LANG_POP([C++])

    AC_MSG_CHECKING([whether $CXX declares __cxa_throw in cxxabi.h])
    AC_LANG_PUSH([C++])
    AC_COMPILE_IFELSE([AC_LANG_SOURCE([
            #include <cxxabi.h>
            void f() { __cxxabiv1::__cxa_throw(0, 0, 0); }
        ])], [
            AC_DEFINE([HAVE_GCC_CXXABI_H_CXA_THROW],[1])
            AC_MSG_RESULT([yes])
        ], [AC_MSG_RESULT([no])])
    AC_LANG_POP([C++])

    AC_MSG_CHECKING([whether $CXX supports __attribute__((warn_unused))])
    AC_LANG_PUSH([C++])
    save_CXXFLAGS=$CXXFLAGS
    CXXFLAGS="$CXXFLAGS -Werror -Wunknown-pragmas"
    AC_COMPILE_IFELSE([AC_LANG_SOURCE([
            struct __attribute__((warn_unused)) dummy {};
        ])], [
            AC_DEFINE([HAVE_GCC_ATTRIBUTE_WARN_UNUSED],[1])
            AC_MSG_RESULT([yes])
        ], [AC_MSG_RESULT([no])])
    CXXFLAGS=$save_CXXFLAGS
    AC_LANG_POP([C++])

    AC_MSG_CHECKING([whether STL uses __attribute__((warn_unused))])
    AC_LANG_PUSH([C++])
    save_CXXFLAGS=$CXXFLAGS
    CXXFLAGS="$CXXFLAGS -Werror -Wunused"
    AC_COMPILE_IFELSE([AC_LANG_SOURCE([
            #include <string>
            void f() { std::string s; }
        ])], [
            AC_MSG_RESULT([no])
        ], [
            AC_DEFINE([HAVE_GCC_ATTRIBUTE_WARN_UNUSED_STL],[1])
            AC_MSG_RESULT([yes])])
    CXXFLAGS=$save_CXXFLAGS
    AC_LANG_POP([C++])
fi

AC_SUBST(HAVE_GCC_AVX)
AC_SUBST(HAVE_GCC_BUILTIN_ATOMIC)

dnl ===================================================================
dnl Identify the C++ library
dnl ===================================================================

AC_MSG_CHECKING([What the C++ library is])
AC_LANG_PUSH([C++])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
#include <utility>
#ifndef __GLIBCXX__
foo bar
#endif
]])],
    [CPP_LIBRARY=GLIBCXX
     cpp_library_name="GNU libstdc++"
    ],
    AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
#include <utility>
#ifndef _LIBCPP_VERSION
foo bar
#endif
]])],
    [CPP_LIBRARY=LIBCPP
     cpp_library_name="LLVM libc++"
    ],
    AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
#include <utility>
#ifndef _MSC_VER
foo bar
#endif
]])],
    [CPP_LIBRARY=MSVCRT
     cpp_library_name="Microsoft"
    ],
    AC_MSG_ERROR([Could not figure out what C++ library this is]))))
AC_MSG_RESULT([$cpp_library_name])
AC_LANG_POP([C++])
AC_SUBST(CPP_LIBRARY)

dnl ===================================================================
dnl C++11
dnl ===================================================================

CXXFLAGS_CXX11=
if test "$COM" = MSC; then
    AC_MSG_CHECKING([whether $CXX supports C++11])
    AC_MSG_RESULT(yes)
    # MSVC supports (a subset of) CXX11 without any switch
    CXXFLAGS_CXX11=
elif test "$GCC" = "yes"; then
    HAVE_CXX11=
    CXXFLAGS_CXX11=
    AC_MSG_CHECKING([whether $CXX supports C++11])
    for flag in -std=gnu++11 -std=gnu++0x -std=c++11 -std=c++0x ; do
        save_CXXFLAGS=$CXXFLAGS
        CXXFLAGS="$CXXFLAGS $flag -Werror"
        AC_LANG_PUSH([C++])
        AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[void f() {}]])],[CXXFLAGS_CXX11=$flag])
        AC_LANG_POP([C++])
        CXXFLAGS=$save_CXXFLAGS
        if test -n "$CXXFLAGS_CXX11"; then
            HAVE_CXX11=TRUE
            break
        fi
    done
    if test "$HAVE_CXX11" = TRUE; then
        AC_MSG_RESULT([yes ($CXXFLAGS_CXX11)])
    else
        AC_MSG_ERROR(no)
    fi
fi

dnl Test for temporarily incompatible libstdc++ 4.7.{0,1}, where
dnl <https://gcc.gnu.org/viewcvs/gcc?view=revision&revision=179528> introduced
dnl an additional member _M_size into C++11 std::list towards 4.7.0 and
dnl <https://gcc.gnu.org/viewcvs/gcc?view=revision&revision=189186> removed it
dnl again towards 4.7.2:
if test $CPP_LIBRARY = GLIBCXX; then
    AC_MSG_CHECKING([whether using C++11 causes libstdc++ 4.7.0/4.7.1 ABI breakage])
    AC_LANG_PUSH([C++])
    AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
#include <list>
#if !defined __GLIBCXX__ || (__GLIBCXX__ != 20120322 && __GLIBCXX__ != 20120614)
    // according to <https://gcc.gnu.org/onlinedocs/libstdc++/manual/abi.html>:
    //   GCC 4.7.0: 20120322
    //   GCC 4.7.1: 20120614
    // and using a range check is not possible as the mapping between
    // __GLIBCXX__ values and GCC versions is not monotonic
/* ok */
#else
abi broken
#endif
        ]])], [AC_MSG_RESULT(no, ok)],
        [AC_MSG_ERROR(yes)])
    AC_LANG_POP([C++])
fi

AC_MSG_CHECKING([whether $CXX supports C++11 without Language Defect 757])
save_CXXFLAGS=$CXXFLAGS
CXXFLAGS="$CXXFLAGS $CXXFLAGS_CXX11"
AC_LANG_PUSH([C++])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
#include <stddef.h>

template <typename T, size_t S> char (&sal_n_array_size( T(&)[S] ))[S];

namespace
{
        struct b
        {
                int i;
                int j;
        };
}
]], [[
struct a
{
        int i;
        int j;
};
a thinga[]={{0,0}, {1,1}};
b thingb[]={{0,0}, {1,1}};
size_t i = sizeof(sal_n_array_size(thinga));
size_t j = sizeof(sal_n_array_size(thingb));
return !(i != 0 && j != 0);
]])
    ], [ AC_MSG_RESULT(yes) ],
    [ AC_MSG_ERROR(no)])
AC_LANG_POP([C++])
CXXFLAGS=$save_CXXFLAGS

AC_MSG_CHECKING([whether $CXX supports C++11 without __float128 compile error])
save_CXXFLAGS=$CXXFLAGS
CXXFLAGS="$CXXFLAGS $CXXFLAGS_CXX11"
AC_LANG_PUSH([C++])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
#include <vector>
    // some Clang fail when compiling against libstdc++ headers with -std=gnu++0x
    // (__float128)
]])
    ],[ AC_MSG_RESULT(yes) ],
    [
        AC_MSG_RESULT(no)
        # The only reason why libstdc++ headers fail with Clang in C++11 mode is because
        # they use the __float128 type that Clang doesn't know (libstdc++ checks whether
        # __float128 is available during its build, but it's usually built using GCC,
        # and so c++config.h hardcodes __float128 being supported). As the only place
        # where __float128 is actually used is in a template specialization,
        # -D__float128=void will avoid the problem there while still causing a problem
        # if somebody actually uses the type.
        AC_MSG_CHECKING([whether -D__float128=void workaround helps])
        CXXFLAGS="$CXXFLAGS -D__float128=void"
        AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
#include <vector>
    // some Clang fail when compiling against libstdc++ headers with -std=gnu++0x
    // (__float128)
]])
    ],
    [
     AC_MSG_RESULT(yes)
     CXXFLAGS_CXX11="$CXXFLAGS_CXX11 -D__float128=void"
    ],
    [
     AC_MSG_ERROR(no)
    ])
])

AC_LANG_POP([C++])
CXXFLAGS=$save_CXXFLAGS

AC_SUBST(CXXFLAGS_CXX11)

dnl ==================================
dnl Check for C++11 "= delete" support
dnl ==================================

AC_MSG_CHECKING([whether $CXX supports C++11 = delete syntax])
save_CXXFLAGS=$CXXFLAGS
CXXFLAGS="$CXXFLAGS $CXXFLAGS_CXX11"
AC_LANG_PUSH([C++])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
struct A
    {
    void test() = delete;
    };
]])],[HAVE_CXX11_DELETE=TRUE],[])

AC_LANG_POP([C++])
CXXFLAGS=$save_CXXFLAGS
if test "$HAVE_CXX11_DELETE" = "TRUE"; then
    AC_MSG_RESULT([yes])
    AC_DEFINE([HAVE_CXX11_DELETE])
else
    AC_MSG_RESULT([no])
fi

dnl ==================================
dnl Check for C++11 "override" support
dnl ==================================

AC_MSG_CHECKING([whether $CXX supports C++11 "override" syntax])
save_CXXFLAGS=$CXXFLAGS
CXXFLAGS="$CXXFLAGS $CXXFLAGS_CXX11"
AC_LANG_PUSH([C++])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
struct A
{
        virtual void test();
};

struct B : A
{
        virtual void test() override;
};
]])],[HAVE_CXX11_OVERRIDE=TRUE],[])

AC_LANG_POP([C++])
CXXFLAGS=$save_CXXFLAGS
if test "$HAVE_CXX11_OVERRIDE" = "TRUE"; then
    AC_MSG_RESULT([yes])
    AC_DEFINE([HAVE_CXX11_OVERRIDE])
else
    AC_MSG_RESULT([no])
fi

dnl ==================================
dnl Check for C++11 "final" support
dnl ==================================

AC_MSG_CHECKING([whether $CXX supports C++11 "final" syntax])
save_CXXFLAGS=$CXXFLAGS
CXXFLAGS="$CXXFLAGS $CXXFLAGS_CXX11"
AC_LANG_PUSH([C++])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
// First check that this correct program that uses "final" compiles
struct A final
{
};

struct B
{
        virtual void test();
};

struct C : B
{
        void test() final;
};
]])],[have_final=yes],[])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
// Then check that the "final" works as expected,
// that this program fails to compile
struct A final
{
};

struct B : A
{
};
]])],[],[final_class_works=yes])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
// Also this should fail to compile
struct B
{
        virtual void test();
};

struct C : B
{
        void test() final;
};

struct D : C
{
        void test();
};
]])],[],[final_method_works=yes])
AC_LANG_POP([C++])

CXXFLAGS=$save_CXXFLAGS

if test "$have_final" = yes -a "$final_class_works" = yes -a "$final_method_works" = yes; then
    AC_MSG_RESULT([yes])
    AC_DEFINE([HAVE_CXX11_FINAL])
else
    AC_MSG_RESULT([no])
fi

dnl ===================================================================
dnl Check for C++11 perfect forwarding support
dnl ===================================================================
AC_MSG_CHECKING([whether $CXX supports C++11 perfect forwarding])
save_CXXFLAGS=$CXXFLAGS
CXXFLAGS="$CXXFLAGS $CXXFLAGS_CXX11"
AC_LANG_PUSH([C++])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
    #include <utility>
    template<typename T, typename... Args> T * f(Args &&... v) {
        return new T(std::forward<Args>(v)...);
    }
    ]], [[
    f<int>(0);
    ]])], [perfect_forwarding=yes], [perfect_forwarding=no])
AC_LANG_POP([C++])
CXXFLAGS=$save_CXXFLAGS
AC_MSG_RESULT([$perfect_forwarding])
if test "$perfect_forwarding" = yes; then
    AC_DEFINE([HAVE_CXX11_PERFECT_FORWARDING])
fi

HAVE_GCC_PRAGMA_OPERATOR=
dnl _Pragma support (may require C++11)
if test "$GCC" = "yes"; then
    AC_MSG_CHECKING([whether $CXX supports _Pragma operator])
    AC_LANG_PUSH([C++])
    save_CXXFLAGS=$CXXFLAGS
    CXXFLAGS="$CXXFLAGS $CXXFLAGS_CXX11 -Werror"
    AC_COMPILE_IFELSE([AC_LANG_SOURCE([
            _Pragma("GCC diagnostic ignored \"-Wformat\"")
        ])], [
            AC_DEFINE([HAVE_GCC_PRAGMA_OPERATOR],[1])
            HAVE_GCC_PRAGMA_OPERATOR=1
            AC_MSG_RESULT([yes])
        ], [AC_MSG_RESULT([no])])
    AC_LANG_POP([C++])
    CXXFLAGS=$save_CXXFLAGS
fi
AC_SUBST(HAVE_GCC_PRAGMA_OPERATOR)

dnl ===================================================================
dnl system stl sanity tests
dnl ===================================================================
HAVE_GCC_VISIBILITY_BROKEN=
if test \( "$_os" != "WINNT" -o "$WITH_MINGW" = "yes" \); then

    AC_LANG_PUSH([C++])

    save_CPPFLAGS="$CPPFLAGS"
    if test -n "$MACOSX_SDK_PATH"; then
        CPPFLAGS="-isysroot $MACOSX_SDK_PATH $CPPFLAGS"
    fi

    # Assume visibility is not broken with libc++. The below test is very much designed for libstdc++
    # only.
    if test "$HAVE_GCC_VISIBILITY_FEATURE" = "TRUE" -a "$CPP_LIBRARY" = GLIBCXX; then
        dnl gcc#19664, gcc#22482, rhbz#162935
        AC_MSG_CHECKING([if STL headers are visibility safe (GCC bug 22482)])
        AC_EGREP_HEADER(visibility push, string, stlvisok=yes, stlvisok=no)
        AC_MSG_RESULT([$stlvisok])
        if test "$stlvisok" = "no"; then
            AC_MSG_WARN([Your libstdc++ headers are not visibility safe. Disabling visibility])
            add_warning "Your libstdc++ headers are not visibility safe. Disabling visibility"
            unset HAVE_GCC_VISIBILITY_FEATURE
        fi
    fi

    if test "$HAVE_GCC_VISIBILITY_FEATURE" = "TRUE"; then
        sharedlink_ldflags_save=$LDFLAGS
        LDFLAGS="$LDFLAGS -fvisibility-inlines-hidden $PICSWITCH $LINKFLAGSSHL"

        AC_MSG_CHECKING([if gcc is -fvisibility-inlines-hidden safe with STL headers])
        AC_LINK_IFELSE([AC_LANG_PROGRAM([[
#include <sstream>
using namespace std;
            ]], [[
istringstream strm( "test" ); return 0;
            ]])],
            # Ugh, surely bad to assume an error message will contain
            # the word "unresolvable", a problem with
            # -fvisibility-inlines-hidden and STL headers might cause
            # some more obscure message on some platform, and anway,
            # the error message could be localised.
            [$EGREP -q unresolvable conftest.err;
            if test $? -eq 0; then gccvisok=no; else gccvisok=yes; fi],[gccvisok=no
        ])
        AC_MSG_RESULT([$gccvisok])
        if test "$gccvisok" = "no"; then
            AC_MSG_WARN([Your gcc is not -fvisibility-inlines-hidden safe, disabling that.])
            add_warning "Your gcc is not -fvisibility-inlines-hidden safe, disabling that."
            HAVE_GCC_VISIBILITY_BROKEN="TRUE"
        fi

        LDFLAGS=$sharedlink_ldflags_save
    fi

    # As the below test checks things when linking self-compiled dynamic libraries, it presumably is irrelevant
    # when we don't make any dynamic libraries?
    if test "$DISABLE_DYNLOADING" = "" -a "$HAVE_GCC_VISIBILITY_FEATURE" = "TRUE" -a "$HAVE_GCC_VISIBILITY_BROKEN" != "TRUE"; then
        AC_MSG_CHECKING([if gcc is -fvisibility-inlines-hidden safe (Clang bug 11250)])
        cat > conftestlib1.cc <<_ACEOF
template<typename T> struct S1 { virtual ~S1() {} virtual void f() {} };
struct S2: S1<int> { virtual ~S2(); };
S2::~S2() {}
_ACEOF
        cat > conftestlib2.cc <<_ACEOF
template<typename T> struct S1 { virtual ~S1() {} virtual void f() {} };
struct S2: S1<int> { virtual ~S2(); };
struct S3: S2 { virtual ~S3(); }; S3::~S3() {}
_ACEOF
        gccvisinlineshiddenok=yes
        if ! $CXX $CXXFLAGS $CPPFLAGS $LINKFLAGSSHL -fPIC -fvisibility-inlines-hidden conftestlib1.cc -o libconftest1$DLLPOST >/dev/null 2>&5; then
            gccvisinlineshiddenok=no
        else
            dnl At least Clang -fsanitize=address and -fsanitize=undefined are
            dnl known to not work with -z defs (unsetting which makes the test
            dnl moot, though):
            my_linkflagsnoundefs=$LINKFLAGSNOUNDEFS
            if test "$COM_GCC_IS_CLANG" = TRUE; then
                for i in $CXX; do
                    case $i in
                    -fsanitize=*)
                        my_linkflagsnoundefs=
                        break
                        ;;
                    esac
                done
            fi
            if ! $CXX $CXXFLAGS $CPPFLAGS $LINKFLAGSSHL -fPIC -fvisibility-inlines-hidden conftestlib2.cc -L. -lconftest1 $my_linkflagsnoundefs -o libconftest2$DLLPOST >/dev/null 2>&5; then
                gccvisinlineshiddenok=no
            fi
        fi

        rm -fr libconftest*
        AC_MSG_RESULT([$gccvisinlineshiddenok])
        if test "$gccvisinlineshiddenok" = "no"; then
            AC_MSG_WARN([Your gcc/clang is not -fvisibility-inlines-hidden safe, disabling that.])
            add_warning "Your gcc/clang is not -fvisibility-inlines-hidden safe, disabling that."
            HAVE_GCC_VISIBILITY_BROKEN="TRUE"
        fi
    fi

    if test "$HAVE_GCC_VISIBILITY_FEATURE" = "TRUE"; then
        AC_MSG_CHECKING([if gcc has a visibility bug with class-level attributes (GCC bug 26905)])
        cat >visibility.cxx <<_ACEOF
#pragma GCC visibility push(hidden)
struct __attribute__ ((visibility ("default"))) TestStruct {
  static void Init();
};
__attribute__ ((visibility ("default"))) void TestFunc() {
  TestStruct::Init();
}
_ACEOF
        if ! $CXX $CXXFLAGS $CPPFLAGS -fpic -S visibility.cxx 2>/dev/null > /dev/null; then
            gccvisbroken=yes
        else
            case "$host_cpu" in
            i?86|x86_64)
                if test "$_os" = "Darwin" -o "$WITH_MINGW" = "yes"; then
                    gccvisbroken=no
                else
                    if $EGREP -q '@PLT|@GOT' visibility.s; then
                        gccvisbroken=no
                    else
                        gccvisbroken=yes
                    fi
                fi
                ;;
            *)
                gccvisbroken=no
                ;;
            esac
        fi
        rm -f visibility.s visibility.cxx

        AC_MSG_RESULT([$gccvisbroken])
        if test "$gccvisbroken" = "yes"; then
            AC_MSG_WARN([Your gcc is not -fvisibility=hidden safe. Disabling visibility])
            add_warning "Your gcc is not -fvisibility=hidden safe. Disabling visibility"
            unset HAVE_GCC_VISIBILITY_FEATURE
        fi
    fi

    CPPFLAGS="$save_CPPFLAGS"

    AC_LANG_POP([C++])
fi

AC_SUBST(HAVE_GCC_VISIBILITY_FEATURE)
AC_SUBST(HAVE_GCC_VISIBILITY_BROKEN)

dnl ***************************************
dnl testing libc version for Linux...
dnl ***************************************
if test "$_os" = "Linux"; then
    AC_MSG_CHECKING([whether libc is >= 2.1.1])
    exec 6>/dev/null # no output
    AC_CHECK_LIB(c, gnu_get_libc_version, HAVE_LIBC=yes; export HAVE_LIBC)
    exec 6>&1 # output on again
    if test "$HAVE_LIBC"; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_ERROR([no, upgrade libc])
    fi
fi

dnl ***************************************
dnl Checking for bison and flex
dnl ***************************************
AC_PATH_PROG(BISON, bison)
if test -z "$BISON"; then
    AC_MSG_ERROR([no bison found in \$PATH, install it])
else
    AC_MSG_CHECKING([the bison version])
    _bison_version=`$BISON --version | grep GNU | $SED -e 's@^[[^0-9]]*@@' -e 's@ .*@@' -e 's@,.*@@'`
    _bison_longver=`echo $_bison_version | $AWK -F. '{ print \$1*1000+\$2}'`
    # Accept newer than 2.0
    if test "$_bison_longver" -lt 2000; then
        AC_MSG_ERROR([failed ($BISON $_bison_version need 2.0+)])
    fi
fi

AC_PATH_PROG(FLEX, flex)
if test "$GNUMAKE_WIN_NATIVE" = "TRUE" ; then
    FLEX=`cygpath -m $FLEX`
fi
if test -z "$FLEX"; then
    AC_MSG_ERROR([no flex found in \$PATH, install it])
else
    AC_MSG_CHECKING([the flex version])
    _flex_version=$($FLEX --version | $SED -e 's/^.*\([[[:digit:]]]\{1,\}\.[[[:digit:]]]\{1,\}\.[[[:digit:]]]\{1,\}\).*$/\1/')
    if test $(echo $_flex_version | $AWK -F. '{printf("%d%03d%03d", $1, $2, $3)}') -lt 2005035; then
        AC_MSG_ERROR([failed ($FLEX $_flex_version found, but need at least 2.5.35)])
    fi
fi
AC_SUBST([FLEX])
dnl ***************************************
dnl Checking for patch
dnl ***************************************
AC_PATH_PROG(PATCH, patch)
if test -z "$PATCH"; then
    AC_MSG_ERROR(["patch" not found in \$PATH, install it])
fi

dnl On Solaris, FreeBSD or MacOS X, check if --with-gnu-patch was used
if test "$_os" = "SunOS" -o "$_os" = "FreeBSD" -o "$_os" = "Darwin"; then
    if test -z "$with_gnu_patch"; then
        GNUPATCH=$PATCH
    else
        if test -x "$with_gnu_patch"; then
            GNUPATCH=$with_gnu_patch
        else
            AC_MSG_ERROR([--with-gnu-patch did not point to an executable])
        fi
    fi

    AC_MSG_CHECKING([whether $GNUPATCH is GNU patch])
    if $GNUPATCH --version | grep "Free Software Foundation" >/dev/null 2>/dev/null; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_ERROR([no, GNU patch needed. install or specify with --with-gnu-patch=/path/to/it])
    fi
else
    GNUPATCH=$PATCH
fi

if test "$GNUMAKE_WIN_NATIVE" = "TRUE" ; then
    GNUPATCH=`cygpath -m $GNUPATCH`
fi

dnl We also need to check for --with-gnu-cp

if test -z "$with_gnu_cp"; then
    # check the place where the good stuff is hidden on Solaris...
    if test -x /usr/gnu/bin/cp; then
        GNUCP=/usr/gnu/bin/cp
    else
        AC_PATH_PROGS(GNUCP, gnucp cp)
    fi
    if test -z $GNUCP; then
        AC_MSG_ERROR([Neither gnucp nor cp found. Install GNU cp and/or specify --with-gnu-cp=/path/to/it])
    fi
else
    if test -x "$with_gnu_cp"; then
        GNUCP=$with_gnu_cp
    else
        AC_MSG_ERROR([--with-gnu-cp did not point to an executable])
    fi
fi

if test "$GNUMAKE_WIN_NATIVE" = "TRUE" ; then
    GNUCP=`cygpath -m $GNUCP`
fi

AC_MSG_CHECKING([whether $GNUCP is GNU cp from coreutils with preserve= support])
if $GNUCP --version 2>/dev/null | grep "coreutils" >/dev/null 2>/dev/null; then
    AC_MSG_RESULT([yes])
elif $GNUCP --version 2>/dev/null | grep "GNU fileutils" >/dev/null 2>/dev/null; then
    AC_MSG_RESULT([yes])
else
    case "$build_os" in
    darwin*|netbsd*|openbsd*|freebsd*|dragonfly*|aix*)
        x_GNUCP=[\#]
        GNUCP=''
        AC_MSG_RESULT([no gnucp found - using the system's cp command])
        ;;
    *)
        AC_MSG_ERROR([no, GNU cp needed. install or specify with --with-gnu-cp=/path/to/it])
        ;;
    esac
fi

AC_SUBST(GNUPATCH)
AC_SUBST(GNUCP)
AC_SUBST(x_GNUCP)

dnl ***************************************
dnl testing assembler path
dnl ***************************************
ML_EXE=""
if test "$_os" = "WINNT" -a "$WITH_MINGW" != "yes"; then
    if test "$BITNESS_OVERRIDE" = ""; then
        assembler=ml.exe
        assembler_bin=bin
    else
        assembler=ml64.exe
        assembler_bin=bin/amd64
    fi

    AC_MSG_CHECKING([$VC_PRODUCT_DIR/$assembler_bin/$assembler])
    if test -f "$VC_PRODUCT_DIR/$assembler_bin/$assembler"; then
        ASM_HOME=$VC_PRODUCT_DIR/$assembler_bin
        AC_MSG_RESULT([found])
        ML_EXE="$VC_PRODUCT_DIR/$assembler_bin/$assembler"
    else
        AC_MSG_ERROR([Configure did not find $assembler assembler.])
    fi

    PathFormat "$ASM_HOME"
    ASM_HOME="$formatted_path"
else
    ASM_HOME=""
fi

AC_SUBST(ML_EXE)

# ===================================================================
# De- or increase default verbosity of build process
# ===================================================================
AC_MSG_CHECKING([build verbosity])
if test -n "$enable_verbose"; then
    if test "$enable_verbose" = "yes"; then
        VERBOSE="TRUE"
        AC_MSG_RESULT([high])
    fi
    if test "$enable_verbose" = "no"; then
        VERBOSE="FALSE"
        AC_MSG_RESULT([low])
    fi
else
    AC_MSG_RESULT([not set])
fi
AC_SUBST(VERBOSE)

dnl ===================================================================
dnl Hook up LibreOffice's nodep environmental variable to automake's equivalent
dnl --enable-dependency-tracking configure option
dnl ===================================================================
AC_MSG_CHECKING([whether to enable dependency tracking])
if test "$enable_dependency_tracking" = "no"; then
    nodep=TRUE
    AC_MSG_RESULT([no])
else
    AC_MSG_RESULT([yes])
fi
AC_SUBST(nodep)

dnl ===================================================================
dnl Number of CPUs to use during the build
dnl ===================================================================
AC_MSG_CHECKING([for number of processors to use])
# plain --with-parallelism is just the default
if test -n "$with_parallelism" -a "$with_parallelism" != "yes"; then
    if test "$with_parallelism" = "no"; then
        PARALLELISM=0
    else
        PARALLELISM=$with_parallelism
    fi
else
    if test "$enable_icecream" = "yes"; then
        PARALLELISM="10"
    else
        case `uname -s` in

        Darwin|FreeBSD|NetBSD|OpenBSD)
            PARALLELISM=`sysctl -n hw.ncpu`
            ;;

        Linux)
            PARALLELISM=`getconf _NPROCESSORS_ONLN`
        ;;
        # what else than above does profit here *and* has /proc?
        *)
            PARALLELISM=`grep $'^processor\t*:' /proc/cpuinfo | wc -l`
            ;;
        esac

        # If we hit the catch-all case, but /proc/cpuinfo doesn't exist or has an
        # unexpected format, 'wc -l' will have returned 0 (and we won't use -j at all).
    fi
fi

if test "$no_parallelism_make" = "YES" && test $PARALLELISM -gt 1; then
    if test -z "$with_parallelism"; then
            AC_MSG_WARN([gmake 3.81 crashes with parallelism > 1, reducing it to 1. upgrade to 3.82 to avoid this.])
            add_warning "gmake 3.81 crashes with parallelism > 1, reducing it to 1. upgrade to 3.82 to avoid this."
            PARALLELISM="1"
    else
        add_warning "make 3.81 is prone to crashes with parallelism > 1. Since --with-parallelism was explicitly given, it is honored, but do not complain when make segfaults on you."
    fi
fi

if test $PARALLELISM -eq 0; then
    AC_MSG_RESULT([explicit make -j option needed])
else
    AC_MSG_RESULT([$PARALLELISM])
fi
AC_SUBST(PARALLELISM)

#
# Set up ILIB for MSVC build
#
if test "$build_os" = "cygwin"; then
    ILIB="."
    if test -n "$JAVA_HOME" -a "$JAVA_HOME" != "NO_JAVA_HOME"; then
        ILIB="$ILIB;$JAVA_HOME/lib"
    fi
    if test "$BITNESS_OVERRIDE" = 64; then
        ILIB="$ILIB;$COMPATH/lib/amd64"
        ILIB="$ILIB;$WINDOWS_SDK_HOME/lib/x64"
        if test $WINDOWS_SDK_VERSION = 80 -o $WINDOWS_SDK_VERSION = 81; then
            ILIB="$ILIB;$WINDOWS_SDK_HOME/lib/$winsdklibsubdir/um/x64"
        fi
    else
        ILIB="$ILIB;$COMPATH/lib"
        ILIB="$ILIB;$WINDOWS_SDK_HOME/lib"
        if test $WINDOWS_SDK_VERSION = 80 -o $WINDOWS_SDK_VERSION = 81; then
            ILIB="$ILIB;$WINDOWS_SDK_HOME/lib/$winsdklibsubdir/um/x86"
        fi
    fi
    ILIB="$ILIB;$DOTNET_FRAMEWORK_HOME/lib"

    if test "$ENABLE_DIRECTX" = "TRUE" -a "$HAVE_DIRECTXSDK_LIB" = "yes"; then
        ILIB="$ILIB;$DIRECTXSDK_LIB"
    fi
    AC_SUBST(ILIB)
fi


AC_SUBST([URELIBS])

dnl ===================================================================
dnl Setting up the environment.
dnl ===================================================================
echo "setting up the build environment variables..."

AC_SUBST(COMPATH)

if test "$build_os" = "cygwin"; then
    PathFormat "/usr/bin/grep.exe"
    WIN_GREP="$formatted_path"
    PathFormat "/usr/bin/find.exe"
    WIN_FIND="$formatted_path"
    PathFormat "/usr/bin/ls.exe"
    WIN_LS="$formatted_path"
    PathFormat "/usr/bin/touch.exe"
    WIN_TOUCH="$formatted_path"
fi

AC_SUBST(WIN_FIND)
AC_SUBST(WIN_GREP)
AC_SUBST(WIN_LS)
AC_SUBST(WIN_TOUCH)

AC_SUBST(BUILD_TYPE)

AC_SUBST(SOLARINC)

PathFormat "$PERL"
PERL="$formatted_path"
AC_SUBST(PERL)

if test -n "$TMPDIR"; then
    TEMP_DIRECTORY="$TMPDIR"
else
    TEMP_DIRECTORY="/tmp"
fi
if test "$build_os" = "cygwin"; then
    TEMP_DIRECTORY=`cygpath -m "$TEMP_DIRECTORY"`
fi
AC_SUBST(TEMP_DIRECTORY)

# setup the PATH for the environment
if test -n "$LO_PATH_FOR_BUILD"; then
    LO_PATH="$LO_PATH_FOR_BUILD"
else
    LO_PATH="$PATH"

    case "$host_os" in

    aix*|dragonfly*|freebsd*|linux-gnu*|*netbsd*|openbsd*)
        if test "$ENABLE_JAVA" != "" -a "$JDK"!="gcj"; then
            pathmunge "$JAVA_HOME/bin" "after"
        fi
        ;;

    cygwin*)
        # Win32 make needs native paths
        if test "$GNUMAKE_WIN_NATIVE" = "TRUE" ; then
            LO_PATH=`cygpath -p -m "$PATH"`
        fi
        pathmunge "$DOTNET_FRAMEWORK_HOME/bin" "before"
        pathmunge "$ASM_HOME" "before"
        pathmunge "$WINDOWS_SDK_HOME/bin" "before"
        pathmunge "$CSC_PATH" "before"
        pathmunge "$MIDL_PATH" "before"
        pathmunge "$AL_PATH" "before"
        pathmunge "$MSPDB_PATH" "before"
        if test -n "$MSBUILD_PATH" ; then
            pathmunge "$MSBUILD_PATH" "before"
        fi
        if test "$BITNESS_OVERRIDE" = 64; then
            pathmunge "$COMPATH/bin/amd64" "before"
            pathmunge "$WINDOWS_SDK_HOME/bin/x64" "before"
        else
            pathmunge "$COMPATH/bin" "before"
            pathmunge "$WINDOWS_SDK_HOME/bin/x86" "before"
        fi
        if test "$ENABLE_JAVA" != "" -a "$JDK"!="gcj"; then
            if test -d "$JAVA_HOME/jre/bin/client"; then
                pathmunge "$JAVA_HOME/jre/bin/client" "before"
            fi
            if test -d "$JAVA_HOME/jre/bin/hotspot"; then
                pathmunge "$JAVA_HOME/jre/bin/hotspot" "before"
            fi
            pathmunge "$JAVA_HOME/bin" "before"
        fi
        ;;

    solaris*)
        pathmunge "/usr/css/bin" "before"
        if test "$ENABLE_JAVA" != "" -a "$JDK"!="gcj"; then
            pathmunge "$JAVA_HOME/bin" "after"
        fi
        ;;
    esac

    pathmunge "$SRC_ROOT/solenv/bin" "before"
    pathmunge "." "before"
fi

AC_SUBST(LO_PATH)

# Generate a configuration md5 we can use for deps
if test -f config_host.mk; then
    config_md5=`$MD5SUM config_host.mk | sed "s/ .*//"`
fi

AC_CONFIG_FILES([config_host.mk
                 Makefile])
AC_OUTPUT

if test "$CROSS_COMPILING" = TRUE; then
    (echo; echo export BUILD_TYPE_FOR_HOST=$BUILD_TYPE) >>config_build.mk
fi

# touch the config timestamp file
if test ! -f config_host.mk.stamp; then
    echo > config_host.mk.stamp
elif test "$config_md5" = `$MD5SUM config_host.mk | sed "s/ .*//"`; then
    echo "Configuration unchanged - avoiding scp2 stamp update"
else
    echo > config_host.mk.stamp
fi


cat << _EOF
****************************************************************************

To build, issue:
$GNUMAKE

For details, run:
$GNUMAKE help

_EOF

if test -f warn; then
    cat warn
    rm warn
fi

dnl vim:set shiftwidth=4 softtabstop=4 expandtab:
