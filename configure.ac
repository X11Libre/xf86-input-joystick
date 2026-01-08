#  Copyright 2005 Adam Jackson.
#
#  Permission is hereby granted, free of charge, to any person obtaining a
#  copy of this software and associated documentation files (the "Software"),
#  to deal in the Software without restriction, including without limitation
#  on the rights to use, copy, modify, merge, publish, distribute, sub
#  license, and/or sell copies of the Software, and to permit persons to whom
#  the Software is furnished to do so, subject to the following conditions:
#
#  The above copyright notice and this permission notice (including the next
#  paragraph) shall be included in all copies or substantial portions of the
#  Software.
#
#  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#  FITNESS FOR A PARTICULAR PURPOSE AND NON-INFRINGEMENT.  IN NO EVENT SHALL
#  ADAM JACKSON BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
#  IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
#  CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#
# Process this file with autoconf to produce a configure script

# Initialize Autoconf
AC_PREREQ([2.60])
AC_INIT([xlibre-xf86-input-joystick],
        [25.0.0],
        [https://github.com/X11Libre/xf86-input-joystick/issues],
        [xlibre-xf86-input-joystick])
AC_CONFIG_SRCDIR([Makefile.am])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_AUX_DIR(.)

# Initialize Automake
AM_INIT_AUTOMAKE([foreign dist-xz])

# Initialize libtool
AC_DISABLE_STATIC
AC_PROG_LIBTOOL

# Initialize X.Org macros 1.8 or later for MAN_SUBSTS set by XORG_MANPAGE_SECTIONS
m4_ifndef([XORG_MACROS_VERSION],
          [m4_fatal([must install xorg-macros 1.8 or later before running autoconf/autogen])])
XORG_MACROS_VERSION(1.8)
XORG_DEFAULT_OPTIONS

PKG_PROG_PKG_CONFIG([0.25])
PKG_CHECK_MODULES(XORG, [xorg-server >= 25.0.0] xproto inputproto kbproto)

moduledir=`$PKG_CONFIG --variable=moduledir xorg-server`
inputdir=${moduledir}/input
AC_SUBST(inputdir)

# Define a configure option to enable code debugging
AC_ARG_ENABLE(debug, AS_HELP_STRING([--disable-debug],
                                    [Disable debugging code (default: enabled)]),
                                    [DEBUGGING=$enableval], [DEBUGGING=yes])
if test "x$DEBUGGING" = xyes; then
   AC_DEFINE(DEBUG, 1, [Enable debugging code])
fi
AM_CONDITIONAL(DEBUG, [test "x$DEBUGGING" = xyes])

# -----------------------------------------------------------------------------
#                Determine which joystick backend to build
# -----------------------------------------------------------------------------
linux_backend=yes
AC_CHECK_HEADERS([linux/joystick.h],, [linux_backend=no])
AM_CONDITIONAL(LINUX_BACKEND, [test "x$linux_backend" = xyes])
if test "x$linux_backend" = xyes; then
    AC_DEFINE(LINUX_BACKEND, 1, [Compile Linux joystick backend])
fi

bsd_backend=no
case $host_os in
*dragonfly*)
  AC_CHECK_HEADERS([bus/u4b/usb.h bus/u4b/usbhid.h], [bsd_backend=yes])
  ;;
*netbsd*)
  bsd_backend=yes
  ;;
*)
  AC_CHECK_HEADERS([dev/usb/usbhid.h], [bsd_backend=yes])
  ;;
esac

AM_CONDITIONAL(BSD_BACKEND, [test "x$bsd_backend" = xyes])
if test "x$bsd_backend" = xyes; then
    AC_CHECK_HEADERS([dev/usb/usb_ioctl.h])
    AC_DEFINE(BSD_BACKEND, 1, [Compile BSD usbhid backend])
fi

evdev_backend=yes
AC_CHECK_HEADERS([linux/input.h],, [evdev_backend=no])
AM_CONDITIONAL(EVDEV_BACKEND, [test "x$evdev_backend" = xyes])
if test "x$evdev_backend" = xyes; then
    AC_DEFINE(EVDEV_BACKEND, 1, [Compile Linux evdev backend])
fi

AC_OUTPUT([Makefile src/Makefile man/Makefile man/joystick.man config/Makefile])

AS_ECHO()
AS_ECHO("Building Linux joystick backend: $linux_backend")
AS_ECHO("Building Linux evdev backend: $evdev_backend")
AS_ECHO("Building BSD usbhid backend: $bsd_backend")

if test "x$linux_backend" != "xyes" -a \
        "x$bsd_backend" != "xyes" -a \
        "x$evdev_backend" != "xyes"; then
    AC_MSG_ERROR([No backends were found. Your operating system is not
                  supported by the joystick driver. Contact
                  xlibre@freelists.org if you are interested in porting it.])
fi
