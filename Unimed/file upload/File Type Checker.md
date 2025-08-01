---
state: "[[Rascunhos]]"
---
# Link

[file-type-checker](https://github.com/nir11/file-type-checker#readme)
# About
Detect and validate file types by their signatures (✨magic numbers✨)
- Supports a wide range of file types.
- Detects the type of a given file using a detection mechanism.
- Validates files against their requested types using signatures.
- Provides flexibility for the file data type: choose from `Array<number>`, `ArrayBuffer`, or `Uint8Array`.
- Written in TypeScript for [[satisfação/deep/full|full]] typing support.
- Works on both server and client sides.
# Example

```js
import fileTypeChecker from "file-type-checker";

// ... Read file as `Array<number>`, `ArrayBuffer`, or `Uint8Array`.

fileTypeChecker.detectFile(file); // Returns the detected file info
fileTypeChecker.validateFileType(file, ["png", "gif", "jpeg"]); // Returns true if the file includes an image signature from the accepted list
fileTypeChecker.isPNG(file); // Returns true if the file includes a valid PNG image signature

// ... And many more validator functions for each supported type.
```

Detect file by its signature (synchronously, will be slow with large files):

```js
import fileTypeChecker from "file-type-checker";
import fs from "fs";

// Read a file as an ArrayBuffer
const file = fs.readFileSync("/path/to/my/file.mp4").buffer;

const detectedFile = fileTypeChecker.detectFile(file);

console.log(detectedFile)
>  {
      "extension": "mp4",
      "mimeType": "video/mp4",
      "description": "A multimedia container format widely used for storing audio, video, and other data, and is known for its high compression efficiency and compatibility with many devices",
      "signature": {
        "sequence": ["66","74","79","70","69","73","6f","6d"],
        "description" (optional): "ISO Base Media file (MPEG-4) v1",
        "offset" (optional): 4
      }
  }
```

Validate file signature against a list of file types:

```js
import fileTypeChecker from "file-type-checker";
import fs from "fs";
//const fileTypeChecker = require("file-type-checker"); // legacy way
//const fs = require("fs"); // legacy way

// Read a file as an ArrayBuffer
const file = fs.readFileSync("/path/to/my/file.png").buffer;

// A list of accepted image file types
const types = ["jpeg", "png", "gif"];

const isImage = fileTypeChecker.validateFileType(
  file,
  types,
  { chunkSize: 32 } // (optional parameter) all images signatures exists in the first 32 bytes
);

console.log(isImage); // Returns true the file includes an image signature from the accepted list
```

Validate file signature against a single file type:

```js
import fileTypeChecker from "file-type-checker";
import fs from "fs";
//const fileTypeChecker = require("file-type-checker"); // legacy way
//const fs = require("fs"); // legacy way

// Read a file as an ArrayBuffer
const file = fs.readFileSync("/path/to/my/file.png").buffer;

const isPNG = fileTypeChecker.isPNG(file);

console.log(isPNG); // Returns true if the file includes a valid PNG image signature
```

## Optimization

### Read only the first X bytes when detecting file type.

- Most file signatures exist in the first 64 bytes, except ZIP files that require 32,000 bytes.
    
- Before calling `detectFile`, `validateFile` or any other validation function, make sure you only pass a chunk from the beginning of the file content.
    
- This reduces unnecessary file I/O and memory consumption.

✅ [[Node]].js example (read first 64 bytes for detection)

```js
import fileTypeChecker from "file-type-checker";
import fs from "fs";
import path from "path";

const file = path.resolve("/path/to/my/large/file.mp4");
const CHUNK_SIZE = 64; // All file signatures except ZIP exist in the first 64 bytes
const fileHandle = await fs.promises.open(file, "r");

const buffer = Buffer.alloc(CHUNK_SIZE);
await fileHandle.read(buffer, 0, CHUNK_SIZE, 0); // Read only the first 64 bytes

// Detect file type using the first chunk
const detectedFileInfo = fileTypeChecker.detectFile(buffer, {
  chunkSize: 64,
});
```

### Process large files in chunks.

- Avoid loading the entire file into memory at once.
    
- Most file signatures exist in the first 64 bytes, except ZIP files that require 32,000 bytes.
    
- Process files in small chunks instead of reading everything at once.

✅ [[Node]].js example (read large files in chunks)

```js
import fileTypeChecker from "file-type-checker";
import fs from "fs";

const readFileInChunks = (filePath, chunkSize = 64 * 1024) => {
  const stream = fs.createReadStream(filePath, { highWaterMark: chunkSize });
  let isFirstChunk = true; // Flag to track the first chunk

  stream.on("data", (chunk) => {
    console.log("Chunk read:", chunk);

    if (isFirstChunk) {
      const detectedFile = fileTypeChecker.detectFile(chunk);
      console.log("Detected File Type:", detectedFile);
      isFirstChunk = false; // Prevent further calls to detectFile
    }
  });

  stream.on("end", () => console.log("Finished reading file."));
};

readFileInChunks("/path/to/my/large/file.mp4");
```

```js
const is7z = fileTypeChecker.is7z(file); // Returns true if the file includes a valid 7z archive signature
const isAAC = fileTypeChecker.isAAC(file); // Returns true if the file includes a valid AAC audio file signature
const isAMR = fileTypeChecker.isAMR(file); // Returns true if the file includes a valid AMR audio file signature
const isAVI = fileTypeChecker.isAVI(file); // Returns true if the file includes a valid AVI video file signature
const isBMP = fileTypeChecker.isBMP(file); // Returns true if the file includes a valid BMP image signature
const isBPG = fileTypeChecker.isBPG(file); // Returns true if the file includes a valid BPG image signature
const isBLEND = fileTypeChecker.isBLEND(file); // Returns true if the file includes a valid Blender 3D file signature
const isCR2 = fileTypeChecker.isCR2(file); // Returns true if the file includes a valid Canon CR2 raw image signature
const isDOC = fileTypeChecker.isDOC(file); // Returns true if the file includes a valid DOC file signature
const isELF = fileTypeChecker.isELF(file); // Returns true if the file includes a valid ELF executable file signature
const isEXR = fileTypeChecker.isEXR(file); // Returns true if the file includes a valid EXR image signature
const isEXE = fileTypeChecker.isEXE(file); // Returns true if the file includes a valid EXE image signature
const isFLAC = fileTypeChecker.isFLAC(file); // Returns true if the file includes a valid FLAC audio file signature
const isFLV = fileTypeChecker.isFLV(file); // Returns true if the file includes a valid FLV video file signature
const isGIF = fileTypeChecker.isGIF(file); // Returns true if the file includes a valid GIF image signature
const isHEIC = fileTypeChecker.isHEIC(file); // Returns true if the file includes a valid HEIC image signature
const isICO = fileTypeChecker.isICO(file); // Returns true if the file includes a valid ICO image signature
const isINDD = fileTypeChecker.isINDD(file); // Returns true if the file includes a valid Adobe InDesign document signature
const isJPEG = fileTypeChecker.isJPEG(file); // Returns true if the file includes a valid JPEG image signature
const isLZH = fileTypeChecker.isLZH(file); // Returns true if the file includes a valid LZH archive signature
const isM4A = fileTypeChecker.isM4A(file); // Returns true if the file includes a valid M4A audio file signature
const isM4V = fileTypeChecker.isM4V(file); // Returns true if the file includes a valid M4V video file signature
const isMACHO = fileTypeChecker.isMACHO(file); // Returns true if the file includes a valid MACH-O video file
const isMKV = fileTypeChecker.isMKV(file); // Returns true if the file includes a valid MKV video file signature
const isMOV = fileTypeChecker.isMOV(file); // Returns true if the file includes a valid MOV video file signature
const isMP3 = fileTypeChecker.isMP3(file); // Returns true if the file includes a valid MP3 audio file signature
const isMP4 = fileTypeChecker.isMP4(file); // Returns true if the file includes a valid MP4 video file signature
const isOGG = fileTypeChecker.isOGG(file); // Returns true if the file includes a valid OGG audio file signature
const isORC = fileTypeChecker.isORC(file); // Returns true if the file includes a valid ORC file signature
const isPARQUET = fileTypeChecker.isPARQUET(file); // Returns true if the file includes a valid Parquet file signature
const isPBM = fileTypeChecker.isPBM(file); // Returns true if the file includes a valid PBM image signature
const isPCAP = fileTypeChecker.isPCAP(file); // Returns true if the file includes a valid PCAP file signature
const isPDF = fileTypeChecker.isPDF(file); // Returns true if the file includes a valid PDF document signature
const isPGM = fileTypeChecker.isPGM(file); // Returns true if the file includes a valid PGM image signature
const isPNG = fileTypeChecker.isPNG(file); // Returns true if the file includes a valid PNG image signature
const isPPM = fileTypeChecker.isPPM(file); // Returns true if the file includes a valid PPM image
const isPSD = fileTypeChecker.isPSD(file); // Returns true if the file includes a valid PSD image signature
const isPS = fileTypeChecker.isPS(file); // Returns true if the file includes a valid PostScript file signature
const isRAR = fileTypeChecker.isRAR(file); // Returns true if the file includes a valid RAR archive signature
const isRTF = fileTypeChecker.isRTF(file); // Returns true if the file includes a valid RTF document signature
const isSQLite = fileTypeChecker.isSQLite(file); // Returns true if the file includes a valid SQLite database file signature
const isSTL = fileTypeChecker.isSTL(file); // Returns true if the file includes a valid STL 3D model file signature
const isSWF = fileTypeChecker.isSWF(file); // Returns true if the file includes a valid SWF file signature
const isTTF = fileTypeChecker.isTTF(file); // Returns true if the file includes a valid TrueType font file signature
const isWAV = fileTypeChecker.isWAV(file); // Returns true if the file includes a valid WAV audio file signature
const isWEBM = fileTypeChecker.isWEBM(file); // Returns true if the file includes a valid WebM video file signature
const isWEBP = fileTypeChecker.isWEBP(file); // Returns true if the file includes a valid WebP image file signature
const isZIP = fileTypeChecker.isZIP(file); // Returns true if the file includes a valid ZIP archive signature
```
## Resources

[GCK File Signature Table](https://filesig.search.org/)

[List of file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)