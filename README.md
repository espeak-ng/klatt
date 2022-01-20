# Klatt Cascade-Parallel Formant Synthesizer

- [Build Dependencies](#build-dependencies)
  - [Debian](#debian)
- [Building](#building)
- [Modifications](#modifications)
- [Input File Format](#input-file-format)
- [Command Line Options](#command-line-options)
- [Example Parameter File](#example-parameter-file)
- [Bugs](#bugs)
- [License Information](#license-information)
- [References](#references)

----------

This is an implementation of the Klatt Cascade-Parallel Formant Speech
Synthesizer. The software for this synthesizer was originally described
in <a href="#klatt1980">Klatt1980</a> and an updated version of the
software was described in <a href="#klatt1990">Klatt1990</a>.

The code was originally written in Fortran by Dennis Klatt. It was ported
to C by John Iles and Nick Ing-Simmons upto version 3.04. The code has since
been modernized and cleaned up to make it easier to build and maintain on
modern C compilers and systems by Reece H. Dunn.

In terms of the two articles referred to above, this version seems to
be the mid point of the development between the two systems described.

## Build Dependencies

In order to build Klatt, you need:

1.  a functional autotools system (`make`, `autoconf` and `automake`);
2.  a functional c compiler.

### Debian

| Dependency | Install                                       |
|------------|-----------------------------------------------|
| autotools  | `sudo apt-get install make autoconf automake` |
| c compiler | `sudo apt-get install gcc`                    |

## Building

Klatt uses a standard autotools build setup. Simply run:

    ./autogen.sh
    ./configure --prefix=/usr
    make
    sudo make install

## Modifications

The main part of the code in this directory was posted to comp.speech
in early 1993 as part of a crude text to speech conversion system. The
code taken from comp.speech seemed to have been modified considerably
from the original, and for use of the synthesizer in research it was
necessary to "fix" the changes that had been made. The major changes
that have been made are:

1. Re-introduced the parallel-only / cascade-parallel switch. This
allows choice of synthesis method, either using both branches, or just
using the parallel branch.

2. Correct use of bandwidth parameters. One of the cascade bandwidth
parameters was being wrongly used in the parallel branch of the
synthesizer.

3. Modified operation of natural voicing source. The amplitude of the
natural voicing source was very much smaller than the amplitude of the
impulse source, making it difficult to swap between them to evaluate
the differences.

4. Removed the software synthesizer from the context of a text to
speech system. The synthesizer is now a stand-alone program, accepting
input as a set of parameters from a file, and allowing output to a
file or to stdout.

5. Increased the number of parameters available for use in the input
file. The original comp.speech software made assumptions about a
number of the control parameters. To provide the greatest flexibility,
these parameters have been made specific in the input file. It is
important to note that the input file format is NOT compatible with
that used by the software originally posted to the comp.speech news group.

6. Added command line options to control the parameters that remain
constant during synthesis.

7. Added F0 flutter control, as described in <a href="#klatt1990">Klatt1990</a>.

8. Subsequently the code in parwave was re-written by Nick to improve
efficiency, and add a more acceptable ANSI style, and generally make
an elegant implementation.

9. Further re-writes have been carried out to remove all global references.
All parameters are passed around in structures.

10. The facility to use a sampled natural excitation waveform has been 
implemented. Naturalness of the resulting synthetic speech can be greatly 
improved by using the glottal excitation waveform from a natural speaker,
especially if it is the speaker on whose voice the synthesis is actually
based. This may be obtained indirectly by inverse-filtering a vowel.

11. This synthesizer appears in modified form as part of Nick's
rsynth text-to-speech system. This may be found at svr-ftp.eng.cam.ac.uk
in comp.speech/sources.

12. Fixed bug to the antiresonator code that caused overflow problems
when compiling on a PC

13. Various minor modifications to ensure correct compilation using
Microsoft C 7.0 (tested) and Borland C (untested). 

14. Modified random number generation for noise production as
previously it was dependent on the size of the "long" type.

## Input File Format

The input file consists of a series of parameter frames. Each frame of
parameters (usually) represents 10ms of audio output, although this
figure can be adjusted down to 1ms per frame. The parameters in each
frame are described below. To avoid confusion, note that the cascade
and parallel branch of the synthesizer duplicate some of the control
parameters. 

| Parameter | Description |
|-----------|-------------|
| f0        | This is the fundamental frequency (pitch) of the utterance in this case it is specified in steps of 0.1 Hz, hence 100Hz will be represented by a value of 1000. |
| av        | Amplitude of voicing for the cascade branch of the synthesizer in dB0. Range 0-70, value usually 60 for a vowel sound. |
| f1        | First formant frequency in 200-1300 Hz. |
| b1        | Cascade branch bandwidth of first formant in the range 40-1000 Hz. |
| f2        | Second formant frequency in the range 550 - 3000 Hz. |
| b2        | Cascade branch bandwidth of second  formant in the range 40-1000 Hz. |
| f3        | Third formant frequency in the range 1200-4999 Hz. |
| b3        | Cascade branch bandwidth of third formant in the range 40-1000 Hz. |
| f4        | Fourth formant frequency in 1200-4999 Hz. |
| b4        | Cascade branch bandwidth of fourth formant in the range 40-1000 Hz. |
| f5        | Fifth formant frequency in the range 1200-4999 Hz. |
| b5        | Cascade branch bandwidth of fifth formant in the range 40-1000 Hz. |
| f6        | Sixth formant frequency in the range 1200-4999 Hz. |
| b6        | Cascade branch bandwidth of sixth formant in the range 40-2000 Hz. |
| fnz       | Frequency of the nasal zero in the range 248-528 Hz. (cascade branch only) |
| bnz       | Bandwidth of the nasal zero in the range 40-1000 Hz. (cascade branch only) |
| fnp       | Frequency of the nasal pole in the range 248-528 Hz. |
| bnp       | Bandwidth of the nasal pole in the range 40-1000 Hz. |
| asp       | Amplitude of aspiration 0-70 dB. |
| kopen     | Open quotient of voicing waveform, range 0-60, usually 30. Will influence the gravelly or smooth quality of the voice. Only works with impulse and antural simulations. For the sampled glottal excitation waveform the open quotient is fixed. |
| aturb     | Amplitude of turbulence 0-80 dB. A value of 40 is useful. Can be used to simulate "breathy" voice quality. |
| tilt      | Spectral tilt in dB, range 0-24. Tilts down the output spectrum. The value refers to dB down at 3Khz. Increasing the value emphasizes the low frequency content of the speech and attenuates the high frequency content. |
| af        | Amplitude of frication in dB, range 0-80. (parallel branch) |
| skew      | Spectral Skew - skewness of alternate periods, range 0-40 |
| a1        | Amplitude of first formant in the parallel branch, in 0-80 dB. |
| b1p       | Bandwidth of the first formant in the parallel branch, in Hz. |
| a2        | Amplitude of parallel branch second formant. |
| b2p       | Bandwidth of parallel branch second formant. |
| a3        | Amplitude of parallel branch third formant. |
| b3p       | Bandwidth of parallel branch third formant. |
| a4        | Amplitude of parallel branch fourth formant. |
| b4p       | Bandwidth of parallel branch fourth formant. |
| a5        | Amplitude of parallel branch fifth formant. |
| b5p       | Bandwidth of parallel branch fifth formant. |
| a6        | Amplitude of parallel branch sixth formant. |
| b6p       | Bandwidth of parallel branch sixth formant. |
| anp       | Amplitude of the parallel branch nasal formant. |
| ab        | Amplitude of bypass frication in dB. 0-80. |
| avp       | Amplitude of voicing for the parallel branch, 0-70 dB. |
| gain      | Overall gain in dB range 0-80. |

## Command Line Options

| Option          | Description |
|-----------------|-------------|
| `-h`            | Displays a help message. |
| `-i <filename>` | Sets input filename. |
| `-o <outfile>`  | Sets output filename. If output filename not specified, stdout is used. |
| `-q`            | Quiet - print no messages. |
| `-t <n>`        | Select output waveform (RTFC !). |
| `-c`            | Select cascade-parallel configuration. Parallel only configuration is default. |
| `-n <number>`   | Number of formants in cascade branch. Default is 5. |
| `-s <n>`        | Set sample rate. Default is 10Khz. |
| `-f <n>`        | Set number of milliseconds per frame. Default is 10ms per frame. |
| `-v <n>`        | Specifies that the voicing source to be used. 1 = impulse train. 2 = natural simulation. 3 = natural samples. Default is natural voicing. |
| `-V <filename>` | Specifies the filename for a sampled natural excitation waveform. See man page for format details. |
| `-r <n>`        | Output raw binary samples, rather than ASCII integer samples. 1 = high byte, low byte arrangement. 2 = low byte, high byte arrangement. |
| `-F <percent>`  | Percentage of f0 flutter. Default is 0. |

## Example Parameter File

Some example parameter files for a short segments of speech are included in
this distribution. e.g. file called example1.par. Use the following
to produce the output waveforms:

    klatt -i example1.par -o example1.dat -f 5 -v 2
    klatt -i example2.par -o example2.dat -f 5 -s 16000 -v 2

The `-r` option can be used to produce raw binary output, which can
then be converted to many different formats using utilities such as
`sox` (sound exchange) which are available from major ftp sites.

An example is given below of conversion to the ulaw encoded format
used by Sun Sparc SLC's

    sox -r 16000 -s -w example.raw -r 8000 -b -U example.au

Beware of the byte ordering of your machine - if the above procedure
produces distored rubbish, try using `-r 2` instead of `-r 1`. This just
reverses the byte ordering in the raw binary output file. It is also
worth noting that the above example reduces the quality of the output,
as the sampling rate is being halved and the number of bits per sample
is being halved. Ideally output should be at 16kHz with 16 bits per
sample. 

## Bugs

Report bugs to the [klatt issues](https://github.com/rhdunn/klatt/issues)
page on GitHub.

## License Information

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 3, or (at your option)
any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program; if not, write to the Free Software
Foundation, Inc., 675 Mass Ave, Cambridge, MA 02139, USA.

## References

<dl>
<dt><a name="klatt1980"></a>Klatt1980</dt>
<dd>Klatt, D.H., <a href="http://www.fon.hum.uva.nl/david/ma_ssp/2010/Klatt-1980-JAS000971.pdf">Software for a cascade/parallel formant synthesizer</a>. Journal of the Acoustical Society of America, pages 971-995, volume 67, number 3. 1980.</dd>
<dt><a name="klatt1990"></a>Klatt1990</dt>
<dd>Klatt, D.H., Klatt, L.C., <a href="http://www.fon.hum.uva.nl/david/ma_ssp/doc/Klatt-1990-JAS000820.pdf">Analysis, synthesis and perception of voice quality variations among female and male talkers</a>. Journal of the Acoustical Society of America, pages 820-857, volume 87, number 2. 1990.</dd>
</dl>
