---
state: "[[Rascunhos]]"
---
# Link

[file-type](https://www.npmjs.com/package/file-type)
# About

> This package is for detecting binary-based file formats, not text-based formats like `.txt`, `.csv`, `.svg`, etc.

## Types

```text
3g2,3gp,3mf,7z,Z,aac,ac3 ,ace ,aif ,alias,amr,ape,apk,apng ,ar  ,arj ,arrow,arw ,asar ,asf ,avi ,avif ,avro ,blend,bmp ,bpg ,bz2 ,cab ,cfb ,chm ,class,cpio ,cr2 ,cr3 ,crx ,cur ,dcm ,deb ,dmg ,dng ,docm ,docx ,dotm ,dotx ,drc ,dsf ,dwg ,elf ,eot ,eps ,epub ,exe ,f4a ,f4b ,f4p ,f4v ,fbx ,flac ,flif ,flv ,gif ,glb ,gz  ,heic ,icc ,icns ,ico ,ics ,indd ,it  ,j2c ,jar ,jls ,jp2 ,jpg ,jpm ,jpx ,jxl ,jxr ,ktx ,lnk ,lz  ,lz4 ,lzh ,m4a ,m4b ,m4p ,m4v ,macho,mid ,mie ,mj2 ,mkv ,mobi ,mov ,mp1 ,mp2 ,mp3 ,mp4 ,mpc ,mpg ,mts ,mxf ,nef ,nes ,odg ,odp ,ods ,odt ,oga ,ogg ,ogm ,ogv ,ogx ,opus ,orf ,otf ,otg ,otp ,ots ,ott ,parqu,pcap ,pdf ,pgp ,png ,potm ,potx ,ppsm ,ppsx ,pptm ,pptx ,ps  ,psd ,pst ,qcp ,raf ,rar ,rm  ,rpm ,rtf ,rw2 ,s3m ,shp ,skp ,spx ,sqlite,stl,swf,tar,tif,ttc,ttf,vcf,voc,vsdx,vtt,wasm,wav,webm,webp,woff,woff2,wv,xcf,xlsm,xlsx,xltm,xltx,xm,xml,xpi,xz ,zip,zst,
```

# Example

## [API](https://www.npmjs.com/package/file-type#api)

- ## [fileTypeFromBuffer(buffer, options)](https://www.npmjs.com/package/file-type#filetypefrombufferbuffer-options)

Detect the file type of a `Uint8Array`, or `ArrayBuffer`.

The file type is detected by checking the [magic number](https://en.wikipedia.org/wiki/Magic_number_\(programming\)#Magic_numbers_in_files) of the buffer.

If file access is available, it is recommended to use `fileTypeFromFile()` instead.

Returns a `Promise` for an object with the detected file type:

- `ext` - One of the [supported file types](https://www.npmjs.com/package/file-type#supported-file-types)
- `mime` - The [MIME type](https://en.wikipedia.org/wiki/Internet_media_type)

Or `undefined` when there is no match.

[buffer](https://www.npmjs.com/package/file-type#buffer)

 - Type: `Uint8Array | ArrayBuffer`

A buffer representing file data. It works best if the buffer contains the entire file. It may work with a smaller portion as well.

```js
import {fileTypeFromBuffer} from 'file-type';
import {readChunk} from 'read-chunk';

const buffer = await readChunk('Unicorn.png', {length: 4100});

console.log(await fileTypeFromBuffer(buffer));
//=> {ext: 'png', mime: 'image/png'}
```



- ### [fileTypeFromFile(filePath, options)](https://www.npmjs.com/package/file-type#filetypefromfilefilepath-options)

Detect the file type of a file path.

This is for [[Node]].js only.

To read from a [`File`](https://developer.mozilla.org/docs/Web/API/File), see [`fileTypeFromBlob()`](https://www.npmjs.com/package/file-type#filetypefromblobblob-options).

The file type is detected by checking the [magic number](https://en.wikipedia.org/wiki/Magic_number_\(programming\)#Magic_numbers_in_files) of the buffer.

Returns a `Promise` for an object with the detected file type:

- `ext` - One of the [supported file types](https://www.npmjs.com/package/file-type#supported-file-types)
- `mime` - The [MIME type](https://en.wikipedia.org/wiki/Internet_media_type)

Or `undefined` when there is no match.

[filePath](https://www.npmjs.com/package/file-type#filepath)

- Type: `string`

The file path to parse.

```js
import {fileTypeFromFile} from 'file-type';

console.log(await fileTypeFromFile('Unicorn.png'));
//=> {ext: 'png', mime: 'image/png'}
```

