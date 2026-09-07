# ffmpeg-x264
This filter encodes raw video to H.264 with x264, driven through the ffmpeg wrapper.

It exists next to [libx264](https://github.com/Bevara/libx264), which drives the
same encoder directly, and the pair is what makes the cost of the wrapper
measurable — see [What the wrapper costs](#what-the-wrapper-costs) below.

## Requirements

[CMake](https://cmake.org/) is used as a build system. To install it, follow
[Debian build instructions](developing_in_debian.md).

[Emscripten SDK](https://emscripten.org/) is required for building
WebAssembly artifacts. To install it, follow the
[Download and Install](https://emscripten.org/docs/getting_started/downloads.html)
guide:

```bash
cd $OPT

# Get the emsdk repo.
git clone https://github.com/emscripten-core/emsdk.git

# Enter that directory.
cd emsdk

# Download and install the latest SDK tools.
./emsdk install latest

# Make the "latest" SDK "active" for the current user. (writes ~/.emscripten file)
./emsdk activate latest
```

## Building the accessor

```bash
# Setup EMSDK and other environment variables. In practice EMSDK is set to be
# $OPT/emsdk.
source $OPT/emsdk/emsdk_env.sh

# Assuming you are in the root level of the cloned repo :
emcmake cmake .
emmake make
```

Once built, you can use and distribute ffmpeg-x264_1.wasm with your universal tags.

## Rebuilding the ffmpeg libraries

`lib/` holds x264 and a static ffmpeg configured with nothing but the encoder
that wraps it. x264 has to be installed first so that ffmpeg's configure finds
it through pkg-config:

```bash
emconfigure $X264_SRC/configure --enable-static --enable-pic --disable-cli \
    --disable-asm --disable-thread --host=i686-gnu --prefix="$OUT"
emmake make install-lib-static

export EM_PKG_CONFIG_PATH=$OUT/lib/pkgconfig
emconfigure $FFMPEG_SRC/configure --target-os=none --arch=x86_32 \
    --enable-cross-compile --disable-x86asm --disable-inline-asm \
    --disable-stripping --disable-programs --disable-doc \
    --disable-runtime-cpudetect --disable-autodetect --disable-pthreads \
    --pkg-config-flags="--static" --nm="$EMSDK/upstream/bin/llvm-nm" \
    --ar=emar --ranlib=emranlib --cc=emcc --cxx=em++ --objcc=emcc --dep-cc=emcc \
    --enable-pic --disable-everything \
    --enable-gpl --enable-libx264 --enable-encoder=libx264
emmake make
```

`--disable-everything` is what makes the accessor a reasonable size: it leaves
libavcodec.a at 1.9 MB where a default `--enable-gpl --enable-libx264` build
produces 82 MB.

## What the wrapper costs

Both accessors drive the same libx264, built from the same source with the same
flags, and were linked from the same tree. The only difference is whether the
encoder is called directly or through `ffenc` and libavcodec:

| Accessor | Filters | Size |
|---|---|---|
| `libx264_1.wasm` | `encx264` | 1 372 776 |
| `ffmpeg-x264_1.wasm` | `ffenc` | 1 704 786 |

**332 010 bytes, 24 %.** That is the whole cost of the wrapper once ffmpeg is
trimmed to the one encoder in use: libavcodec's encoder plumbing, the parts of
libavutil it drags in, and GPAC's `ff_common.c` with the codec, pixel format
and option mapping tables that make an AVCodec look like a GPAC filter.

For scale, the same module built against a default `--enable-gpl
--enable-libx264` ffmpeg - every decoder, encoder, muxer and demuxer ffmpeg
knows - and carrying the `ffmx` muxer filter as well comes to 14 779 621 bytes,
just under eleven times the direct accessor. Almost none of that is the
wrapper; it is the rest of ffmpeg riding along.

The `ffmx` muxer filter that used to be built here has been dropped: with
`--disable-everything` there is no muxer left for it to expose. Re-adding it is
a matter of putting `ff_mx.c` back and adding `--enable-muxer=` to the configure
line above; it accounted for 48 505 of the bytes above.

## Note on defaults

`ffenc` and `encx264` do not encode to the same quality out of the box: `ffenc`
follows libavcodec's default bitrate, `encx264` sets its own. On a 352x288 test
sequence that showed as 47.4 dB against 59.4 dB PSNR. It is a difference of
default settings, not of encoder.

## Documentation

For more details, please visit our documentation at https://bevara.com/documentation/develop/.
