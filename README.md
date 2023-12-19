# SBEMU-X

Emulate Sound Blaster and OPL3 in pure DOS using modern
PCI-based (onboard and add-in card) sound cards.

## Changes compared to sbemu

This is obviously a friendly fork of [SBEMU](https://github.com/crazii/SBEMU)
to allow for cross-compilation, additional driver support,
continuous integration, bug fixes, etc..

Ideally, we'd like to get these changes integrated upstream
eventually.

For now, this fork provides additional support:

* This updated README ([upstream PR#47](https://github.com/crazii/SBEMU/pull/47))
* Nicer text-mode user interface ([upstream PR#50](https://github.com/crazii/SBEMU/pull/50))
* Additional sound card support
  * CMI 8338 / 8738 (untested) ([upstream PR#48](https://github.com/crazii/SBEMU/pull/48))

Things that were in this fork and are now merged upstream:

* Build support under Linux, Windows, macOS (DJGPP cross-compilation) ([upstream PR#28](https://github.com/crazii/SBEMU/pull/28))
* Additional sound card support
  * SIS 7012 ([upstream PR#29](https://github.com/crazii/SBEMU/pull/29))
* CD-Audio (via 4-pin audio header) unmute/mixer support ([upstream PR#36](https://github.com/crazii/SBEMU/pull/36))
* Fix audio routing on AC97 (ICH5) with AD1980 (FSC-E600 D1534 with i865G) ([upstream PR#35](https://github.com/crazii/SBEMU/pull/35))
* Toggleable debug output on serial port (/DBG0, /DBG1, /DBG2) ([upstream PR#44](https://github.com/crazii/SBEMU/pull/44))
* SB Direct Mode improvements ([PR#10](https://github.com/sbemu-x/sbemu-x/pull/10))
* Time constant fixup moved behind /FIXTC flag ([PR#10](https://github.com/sbemu-x/sbemu-x/pull/10))

## Supported sound cards

Source code from [MPXPlay](https://mpxplay.sourceforge.net/)
is used to support the following sound cards/chips.

Enabled and working:

 * `sc_ich`: Intel ICH / nForce / SIS 7012
 * `sc_inthd`: Intel High Definition Audio
 * `sc_via82`: VIA VT82C686, VT8233
 * `sc_sbliv`: SB Live! / Audigy
 * `sc_sbl24`: SB Audigy LS (CA0106)

Support compiled-in, but untested:

 * `sc_es1371`: Ensoniq ES1371/1373
 * `sc_cmi`: C-Media CMI8338/8738

Source code exists, but "doesn't work yet":

 * `sc_sbxfi`: Creative X-Fi EMU20KX

Source code exists, but not sure if viable:

 * `sc_wss`: Windows Sound System compatible cards
 * `sc_cmi`: CMI 8338/8738 (PCI) cards
 * `sc_ess`: ESS card handling
 * `sc_gus`: Gravis UltraSound
 * `sc_midas`: MIDAS library handling

## Emulated modes

 * 8-bit and 16-bit DMA (mono, stereo, high-speed)
 * Sound Blaster 1.0, 2.0, Pro, 16
 * OPL3 FM via [DOSBox' OPL3 FM implementation](https://www.dosbox.com/)

## Requirements

 * [HDPMI32i](https://github.com/crazii/HX) (HDPMI with IOPL0)
 * Optional, for real-mode game support (I/O trapping):
   * [JEMM](https://github.com/Baron-von-Riedesel/Jemm) with QPIEMU.DLL loaded
   * or [QEMM](https://en.wikipedia.org/wiki/QEMM), commercial software

For memory management, use either:

 * `JEMMEX` only: Provides both HIMEM + EMM
 * `HIMEMX` and `JEMM386`: Separate HIMEM + EMM

In both cases, use `JLOAD` (from the Jemm distribution)
to load `QPIEMU.DLL` before starting `SBEMU`,
so that real-mode support is enabled. If you don't load
JEMM+QPIEMU (or QEMM), only protected mode applications
will be supported.

## Building from source

macOS, Linux and Windows is supported. For Windows, consider using
WSL2 + Linux binaries. DOS isn't supported officially, because why
would you use a single-tasking operating system (either emulated or
on old hardware) to build this project? :)

### Installing a cross-compiler (DJGPP)

Scripts to build a recent GCC toolchain for DJGPP are available here:

* https://github.com/andrewwutw/build-djgpp

There's also prebuilt releases for the toolchain if you don't want
to build DJGPP yourself. The current version (October 2023) is using
GCC 12.2.0, but in the future newer GCC versions might become available:

* https://github.com/andrewwutw/build-djgpp/releases

### Installing make

This assumes a Debian/Ubuntu installation. If you are using any other
distro, I'm assuming you know your way around and can translate those
instructions to your specific distribution.

To get `make` and other tools, it's easiest to install host build tools:

   sudo apt install -y build-essential

On MacOS, install the Xcode command-line tools, which should give you
`make` and other host utilities.

If you are planning on building DJGPP from source, some additional build
tools are needed. Refer to the `build-djgpp` README file for details.

### Building the project

The `bin` folder of your DJGPP toolchain needs to be in your `$PATH`,
so that the following command works and outputs your DJGPP GCC version:

    i586-pc-msdosdjgpp-gcc --version

If this works, building the project is as simple as:

    make

Because you are on a modern machine with multi-core CPUs, do a parallel
build, which is faster, for example, for a quad-core CPU, use 8 parallel
processes to speed up building:

    make -j8

After the build is done, you'll find the build result in a folder called
`output`, i.e. `output/sbemu-x.exe`.

## Feature usage

### CD Audio

CD audio support in DOS requires two parts:

1. Audio control (play/pause/seek/...) via `MSCDEX` (or `SHSUCDX`)
2. Volume control via the mixer

For part one, you need to have a CD-ROM drive with analog audio out
and an MSCDEX-compatible CD-ROM driver set up.

Part two (volume control) is taken care of by SBEMU-X on startup.

To adjust the volume of CD-Audio (by default it's 100% volume),
you can use any Sound Blaster-compatible program, such as "SBMIX",
as SBEMU-X does emulate and forward CD-Audio mixer settings.

Don't forget that to actually hear anything, you need to connect
an analog audio cable from your CD-ROM drive to the 4-pin CD-IN
header on your soundcard (or motherboard for onboard sound).


### Debug output on serial

You can configure SBEMU to output its debug messages to the serial
port instead of the console. This also works in the background when
games are full-screen, and so is really useful for debugging.

To build SBEMU-X with debug output, use:

    make DEBUG=1

Then, launch SBEMU-X with this command for debug output (9600, 8N1)
on COM1 (use `/DBG2` for COM2):

    sbemu-x /DBG1

To disable serial port debug output at runtime, use:

    sbemu-x /DBG0

Serial debug output is disabled by default.
