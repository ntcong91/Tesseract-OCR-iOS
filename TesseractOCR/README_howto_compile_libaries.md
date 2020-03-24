
### Step 1 - Prerequisites
First you need to install these tools:

- Xcode 7 with command line tools
- M4
- Autoconf
- Automake
- Libtool

### Step 2 - Build
Run `make` in the `TesseractOCR` subfolder. This first compiles dependent libraries (png, jpeg, tiff, leptonica) and then tesseract for every architecture iOS/simulator uses (arm7 arm7s arm64 i386 x86_64), and then combines the resulting libs into one library file. It does this for both dependent libraries and tesseract, so the final results of the script are "libpng.a", "libjpeg.a", "libtiff.a", "libtesseract.a", "liblept.a", and "include" directories for both leptonica, tesseract and image libraries. Finally, the script copies these results into the "lib" and "include" directories inside `TesseractOCR` directory.

The very first total build (includinf all architectures) may take half an hour or so depending on the processing power, but the later builds will not build dependencies until any files being changed.

By default every "fat" library will contain all architectures specified above. So it can be linked with apps either for devices or simulator. If you don't need all architectures above (for example, for AppStore submittion), just specify the necessary architectures in the `ARCHS` environement variable as follows:

    export ARCHS=armv7, armv7s, arm64

### Build Notes
There were some problems during compilation, I recorded them.

#### Build Leptonica

##### error: 'system' is unavailable: not available on iOS

I upgrade Leptonica version from 1.76.0 to 1.79.0 and fixed.

#### jp2kio.o error

In this case it needed adding `--without-libopenjpeg` to the call to `./configure` for the `leptonica` target in the make file.

#### webp.o error

In this case it needed adding `--without-libwebp` to the call to `./configure` for the `leptonica` target in the make file.

#### Build Tesseract

##### Compilation for SSE, AVX, AVX2, FMA enabled architectures

`tesseract-ocr/configure.ac` detects SSE, AVX, AVX2, FMA support always as true causing related errors in the Tesseract 4.1+ code.
Build without these errors is to change each such related detection from:
```
AX_CHECK_COMPILE_FLAG([-mavx], [avx=true], [avx=false], [$WERROR])
...
AX_CHECK_COMPILE_FLAG([-mavx2], [avx2=true], [avx2=false], [$WERROR])
...
AX_CHECK_COMPILE_FLAG([-mfma], [fma=true], [fma=false], [$WERROR])
...
AX_CHECK_COMPILE_FLAG([-msse4.1], [sse41=true], [sse41=false], [$WERROR])
```

to:

```
AX_CHECK_COMPILE_FLAG([-mavx], [avx=false], [avx=false], [$WERROR])
...
AX_CHECK_COMPILE_FLAG([-mavx2], [avx2=false], [avx2=false], [$WERROR])
...
AX_CHECK_COMPILE_FLAG([-mfma], [fma=false], [fma=false], [$WERROR])
...
AX_CHECK_COMPILE_FLAG([-msse4.1], [sse41=false], [sse41=false], [$WERROR])
```

And I have copied and modified `TesseractOCR/tesseract-ocr/configure.ac` to `TesseractOCR/tesseract-configure.ac`, everytime the script will copy into tesseract-ocr.

##### fatal error: 'curl/curl.h' file not found

Build curl lib.