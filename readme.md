# imagemin-mozjpeg-baremetal

**IMPORTANT:** This is a fork that remove the node mozjpeg dependency and use the system mozjpeg installation that you should do by your self. I made this fork because the orginal was not working in any way with Linux Alpine and I think with other Linux distros too. 

**REQUIREMENTS:** This library is compatible with mozjpeg 3.1.1. You should install it by your own in your system and add it to the PATH env var so can be accesed from any where. See Linux Alpine section for more details.

> [Imagemin](https://github.com/imagemin/imagemin) plugin for [mozjpeg](https://github.com/mozilla/mozjpeg)


## Install

```
$ npm install imagemin-mozjpeg-baremetal
```


## Usage

```js
const imagemin = require('imagemin');
const imageminMozjpeg = require('imagemin-mozjpeg');

(async () => {
	await imagemin(['images/*.jpg'], {
		destination: 'build/images',
		plugins: [
			imageminMozjpeg()
		]
	});

	console.log('Images optimized');
})();
```


## API

### imageminMozjpeg([options])(buffer)

Returns a `Promise<Buffer>`.

#### options

##### quality

Type: `number`

Compression quality, in range `0` (worst) to `100` (perfect).

##### progressive

Type: `boolean`<br>
Default: `true`

`false` creates baseline JPEG file.

##### targa

Type: `boolean`<br>
Default: `false`

Input file is Targa format (usually not needed).

##### revert

Type: `boolean`<br>
Default: `false`

Revert to standard defaults instead of mozjpeg defaults.

##### fastCrush

Type: `boolean`<br>
Default: `false`

Disable progressive scan optimization.

##### dcScanOpt

Type: `number`<br>
Default: `1`

Set DC scan optimization mode.

- `0` One scan for all components
- `1` One scan per component
- `2` Optimize between one scan for all components and one scan for 1st component plus one scan for remaining components

##### trellis

Type: `boolean`<br>
Default: `true`

[Trellis optimization](https://en.wikipedia.org/wiki/Trellis_quantization).

##### trellisDC

Type: `boolean`<br>
Default: `true`

Trellis optimization of DC coefficients.

##### tune

Type: `string`<br>
Default: `hvs-psnr`

Set Trellis optimization method. Available methods: `psnr`, `hvs-psnr`, `ssim`, `ms-ssim`

##### overshoot

Type: `boolean`<br>
Default: `true`

Black-on-white deringing via overshoot.

##### arithmetic

Type: `boolean`<br>
Default: `false`

Use [arithmetic coding](https://en.wikipedia.org/wiki/Arithmetic_coding).

##### dct

Type: `string`<br>
Default: `int`

Set [DCT](https://en.wikipedia.org/wiki/Discrete_cosine_transform) method:

- `int` Use integer DCT
- `fast` Use fast integer DCT (less accurate)
- `float` Use floating-point DCT

##### quantBaseline

Type: `boolean`<br>
Default: `false`

Use 8-bit quantization table entries for baseline JPEG compatibility.

##### quantTable

Type: `number`

Use predefined quantization table.

- `0` JPEG Annex K
- `1` Flat
- `2` Custom, tuned for MS-SSIM
- `3` ImageMagick table by N. Robidoux
- `4` Custom, tuned for PSNR-HVS
- `5` Table from paper by Klein, Silverstein and Carney

##### smooth

Type: `number`

Set the strength of smooth dithered input. (1...100)

##### maxMemory

Type: `number`

Set the maximum memory to use in kilobytes.

##### sample

Type: `string[]`

Set component sampling factors. Each item should be in the format `HxV`, for example `2x1`.

#### buffer

Type: `buffer`

Buffer to optimize.

## Linux Alpine installation

```
FROM node:12.13.0-alpine as ship

# These are needed to compile mozjpeg
RUN apk --update --no-cache add \
		build-base \
		libpng-dev \
		autoconf \
		automake \
		libtool \
		pkgconf \
		nasm \
		tar


# Install mozjpeg
ARG tag=v3.3.1
WORKDIR /src/mozjpeg
ADD https://github.com/mozilla/mozjpeg/archive/$tag.tar.gz ./

RUN tar -xzf $tag.tar.gz && \
    rm $tag.tar.gz && \
    SRC_DIR=$(ls -t1 -d mozjpeg-* | head -n1) && \
    cd $SRC_DIR && \
    autoreconf -fiv && \
    cd .. && \
    sh $SRC_DIR/configure --enable-static --disable-shared --disable-dependency-tracking --with-jpeg8 && \
    make install \
         prefix=/usr/local \
         libdir=/usr/local/lib64

# To make it available from any place 
export PATH="/usr/local/bin:${PATH}"
```


## License

MIT © [Imagemin](https://github.com/imagemin)
