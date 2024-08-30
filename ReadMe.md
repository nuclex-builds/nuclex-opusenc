![Opus Logo](./opus-logo.svg)

Custom LibOpusEnc Build
------------------------

Opus is a free and open audio codec. A pretty amazing one, too,
because it outperforms other lossy codecs in every discipline -
lower latency, higher quality, better compression efficiency.

libopusenc is a C/C++ library that makes it relatively easy to encode
Opus audio files from raw audio data.


What's in this repository?
--------------------------

This `CMakeLists.txt` is a custom build that *blindly* assumes a modern
desktop Linux build environment / Windows Visual Studio environment
and skips all configure checks.

You should prefer the `CMakeLists.txt` that ships with libopusenc.

This one exists specifically for some of my "Nuclex" projects, where
I'm being pedantic about compiler settings. The `CMakeLists.txt` in this
repository takes all C/C++ compiler options from an external CMake include
file (`../../build-system/cmake/cplusplus.cmake`), thus making sure that
all libraries involved in my tools are compiled with the exact same compiler
settings and that i.e. PGO can be used from end to end.
