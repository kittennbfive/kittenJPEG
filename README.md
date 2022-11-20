# kittenJPEG
a minimalistic Baseline JPEG-decoder / JPG-file-reader, less than 800 lines of C and configurable to spit out A LOT of internal data

## What is this?
This is my attempt to write a decoder for Baseline JPEG / JFIF / .jpg-files. It will parse a .jpg file, optionally output a lot of internal informations (from quantization tables to details of the Huffman-encoded data stream) and optionally save the decoded image as a .ppm-file. It is for Linux only but with a few modifications (write your own implementation of `err()` and `errx()`) it should compile on almost any platform.

## Licence and Disclaimer
This code is provided under AGPLv3+ and WITHOUT ANY WARRANTY!  
The example picture `kitten.jpg` (and its resized version `kitten_small.jpg`) was copied from [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Chat_regardant_le_ciel_\(2016_photo;_cropped_2022\).jpg) and is licenced under Creative Commons CC0 1.0 Universal Public Domain Dedication by its author susannp4 (Thank you!).  

## Why did you made this?
Because kittens are curious. :-)  
I wanted to know more about the internals of JP(E)G, but every tool/decoder i found on the web had some limitations like closed source or Windows-only or not working/crashing or not supporting Chroma-subsampling or not showing all the details or ... So i wrote my own implementation in - hopefully - easy to understand C.

## What is this good for?
If you just want to view a jpg-file this is not what you are looking for at all, there are tons of image-viewers for all existing operating systems. However if you want to know more about the internals of JPEG and/or write or debug your own decoder this code could be handy.

## Specifications
This code will work only for Baseline JPEG with Huffman-Encoding, 8 bit precision and YCbCr-data. EXIF-data will be skipped without being decoded, just use something like `exiftool`. This code does support Chroma-subsampling and dimensions that are not a multiple of 8 pixels. As said it can be configured (using `#define`) to print internal data. Please beware that the resulting output, if redirected to a file, can be HUGE (hundreds of MB)!

## How to use
### Configuration
The `main()` at the bottom of the file is just one variable and 4 function calls. You need to adjust the name of the input file and - if you want a .ppm - the name of the output file. If you don't want a .ppm just comment the corresponding function call (ppm-files are obviously huge compared to jpg!).  
At the top of the file there are some `#define` to enable printing of internal details:
#### `PRINT_DETAILS_HUFFMAN_TABLES`
Print the details of the Huffman tables (all codes), not too much text.
#### `PRINT_DETAILS_QUANTTABLES`
Print the quantization tables, little text.
#### `PRINT_DETAILS_BITSTREAM_LOOP_POSITION`
Show decoding progress by printing current MCU (minimum coded unit) and so on, depending on your input file this can give quite a few lines.
#### `PRINT_DETAILS_BITSTREAM_DECODED_DC`
Show decoded DC-values, same thing as above.
#### `PRINT_DETAILS_BITSTREAM_DECODED_AC`
Show decoded AC-values, potentially a lot of text too.
#### `PRINT_DETAILS_BITSTREAM_MATRIX_ZZ`
Show "raw" matrix reconstructed from DC/AC-value(s) before any further processing, potentially a lot of text.
#### `PRINT_DETAILS_BITSTREAM_MATRIX_DEQUANT`
Show matrix after the ZigZag ordering has been reversed and dequantization applied, potentially a lot of text.
#### `PRINT_DETAILS_BITSTREAM_MATRIX_DECODED`
Show matrix after IDCT, potentially a lot of text.
#### `PRINT_DETAILS_HUFFMAN_BITSTREAM`
Print informations about each decoded Huffman-code, even more text.  
  
In doubt just try, you can always stop the running decoder with Ctrl+C.
### Compiling
`gcc -Wall -Wextra -O3 -o kittenJPEG kittenJPEG.c -lm`

## Performance
This code is primarily optimized for readability, however performance is not too bad. Without any additional printouts (all `#define` commented) processing `kitten.jpg` (~8MP) and writing `kitten.ppm` takes less than 5 seconds on my system.

## Example output
### Minimal version (all `#define` commented):
```
2490424 bytes read from kitten.jpg

SOI found

APP0 found (length 16 bytes)
version 1.1
units 1
density X 300 Y 300
no thumbnail

APP1 (probably EXIF) found (length 214 bytes), skipping

DQT found (length 67 bytes)
Pq (element precision) 0 -> 8 bits
Tq (table destination identifier) 0


DQT found (length 67 bytes)
Pq (element precision) 0 -> 8 bits
Tq (table destination identifier) 1


SOF0 found (length 17 bytes)
P 8 (must be 8)
imagesize X 2827 Y 2920
Nf (number of components) 3
component 0 (Y) C 1, H 2, V 2, Tq 0
component 1 (Cb) C 2, H 1, V 1, Tq 1
component 2 (Cr) C 3, H 1, V 1, Tq 1
Hmax 2 Vmax 2
MCU_total 32391
component 0 (Y) xi 2827 yi 2920
component 1 (Cb) xi 1414 yi 1460
component 2 (Cr) xi 1414 yi 1460
allocating mem for pixels
memory allocated

DHT found (length 30 bytes)
Tc 0 (DC table)
Th (table destination identifier) 0
total 11 codes

DHT found (length 81 bytes)
Tc 1 (AC table)
Th (table destination identifier) 0
total 62 codes

DHT found (length 28 bytes)
Tc 0 (DC table)
Th (table destination identifier) 1
total 9 codes

DHT found (length 59 bytes)
Tc 1 (AC table)
Th (table destination identifier) 1
total 40 codes

SOS found (length 12 bytes)
Ns 3
component 0 (Y) Cs 1 Td 0 Ta 0
component 1 (Cb) Cs 2 Td 1 Ta 1
component 2 (Cr) Cs 3 Td 1 Ta 1
Ss 0 Se 63 Ah 0 Al 0
compressed pixeldata starts at pos 613

removing stuffing...
2489809 bytes with stuffing
2480908 data bytes without stuffing

parsing bitstream...
parsed 32391 MCU

EOI found

writing file kitten.ppm
output file written

cleaned up, all fine
```
### Extremely verbose (show everything)
The complete output is 772MB(!) with 1,3M(!) lines of text; obviously i only show some extracts here...
```
2490424 bytes read from kitten.jpg

SOI found

APP0 found (length 16 bytes)
version 1.1
units 1
density X 300 Y 300
no thumbnail

APP1 (probably EXIF) found (length 214 bytes), skipping

DQT found (length 67 bytes)
Pq (element precision) 0 -> 8 bits
Tq (table destination identifier) 0
02 01 01 02 01 01 02 02 
02 02 02 02 02 02 03 05 
03 03 03 03 03 06 04 04 
03 05 07 06 07 07 07 06 
07 07 08 09 11 09 08 08 
10 08 07 07 10 13 10 10 
11 12 12 12 12 07 09 14 
15 13 12 14 11 12 12 12 


DQT found (length 67 bytes)
Pq (element precision) 0 -> 8 bits
Tq (table destination identifier) 1
02 02 02 03 03 03 06 03 
03 06 12 08 07 08 12 12 
12 12 12 12 12 12 12 12 
12 12 12 12 12 12 12 12 
12 12 12 12 12 12 12 12 
12 12 12 12 12 12 12 12 
12 12 12 12 12 12 12 12 
12 12 12 12 12 12 12 12 


SOF0 found (length 17 bytes)
P 8 (must be 8)
imagesize X 2827 Y 2920
Nf (number of components) 3
component 0 (Y) C 1, H 2, V 2, Tq 0
component 1 (Cb) C 2, H 1, V 1, Tq 1
component 2 (Cr) C 3, H 1, V 1, Tq 1
Hmax 2 Vmax 2
MCU_total 32391
component 0 (Y) xi 2827 yi 2920
component 1 (Cb) xi 1414 yi 1460
component 2 (Cr) xi 1414 yi 1460
allocating mem for pixels
memory allocated

DHT found (length 30 bytes)
Tc 0 (DC table)
Th (table destination identifier) 0
length 1 bits: 0 codes
length 2 bits: 1 codes
length 3 bits: 5 codes
length 4 bits: 1 codes
length 5 bits: 1 codes
length 6 bits: 1 codes
length 7 bits: 1 codes
length 8 bits: 1 codes
length 9 bits: 0 codes
length 10 bits: 0 codes
length 11 bits: 0 codes
length 12 bits: 0 codes
length 13 bits: 0 codes
length 14 bits: 0 codes
length 15 bits: 0 codes
length 16 bits: 0 codes
total 11 codes
codeword 00 (0x0) (sz 2) -> 4 (0x4)
codeword 010 (0x2) (sz 3) -> 2 (0x2)
codeword 011 (0x3) (sz 3) -> 3 (0x3)
codeword 100 (0x4) (sz 3) -> 5 (0x5)
codeword 101 (0x5) (sz 3) -> 6 (0x6)
codeword 110 (0x6) (sz 3) -> 7 (0x7)
codeword 1110 (0xe) (sz 4) -> 1 (0x1)
codeword 11110 (0x1e) (sz 5) -> 8 (0x8)
codeword 111110 (0x3e) (sz 6) -> 0 (0x0)
codeword 1111110 (0x7e) (sz 7) -> 9 (0x9)
codeword 11111110 (0xfe) (sz 8) -> 10 (0xa)

DHT found (length 81 bytes)
Tc 1 (AC table)
Th (table destination identifier) 0
length 1 bits: 0 codes
length 2 bits: 2 codes
length 3 bits: 1 codes
length 4 bits: 2 codes
length 5 bits: 4 codes
length 6 bits: 4 codes
length 7 bits: 4 codes
length 8 bits: 3 codes
length 9 bits: 6 codes
length 10 bits: 5 codes
length 11 bits: 4 codes
length 12 bits: 2 codes
length 13 bits: 0 codes
length 14 bits: 2 codes
length 15 bits: 0 codes
length 16 bits: 23 codes
total 62 codes
codeword 00 (0x0) (sz 2) -> 1 (0x1)
codeword 01 (0x1) (sz 2) -> 2 (0x2)
codeword 100 (0x4) (sz 3) -> 3 (0x3)
codeword 1010 (0xa) (sz 4) -> 4 (0x4)
[...]

DHT found (length 28 bytes)
Tc 0 (DC table)
Th (table destination identifier) 1
length 1 bits: 0 codes
length 2 bits: 2 codes
length 3 bits: 3 codes
length 4 bits: 1 codes
length 5 bits: 1 codes
length 6 bits: 1 codes
length 7 bits: 1 codes
length 8 bits: 0 codes
length 9 bits: 0 codes
length 10 bits: 0 codes
length 11 bits: 0 codes
length 12 bits: 0 codes
length 13 bits: 0 codes
length 14 bits: 0 codes
length 15 bits: 0 codes
length 16 bits: 0 codes
total 9 codes
codeword 00 (0x0) (sz 2) -> 2 (0x2)
codeword 01 (0x1) (sz 2) -> 3 (0x3)
codeword 100 (0x4) (sz 3) -> 1 (0x1)
codeword 101 (0x5) (sz 3) -> 4 (0x4)
codeword 110 (0x6) (sz 3) -> 5 (0x5)
codeword 1110 (0xe) (sz 4) -> 0 (0x0)
codeword 11110 (0x1e) (sz 5) -> 6 (0x6)
codeword 111110 (0x3e) (sz 6) -> 7 (0x7)
codeword 1111110 (0x7e) (sz 7) -> 8 (0x8)

DHT found (length 59 bytes)
Tc 1 (AC table)
Th (table destination identifier) 1
length 1 bits: 0 codes
length 2 bits: 2 codes
length 3 bits: 2 codes
length 4 bits: 2 codes
length 5 bits: 2 codes
length 6 bits: 2 codes
length 7 bits: 1 codes
length 8 bits: 3 codes
length 9 bits: 2 codes
length 10 bits: 4 codes
length 11 bits: 5 codes
length 12 bits: 4 codes
length 13 bits: 2 codes
length 14 bits: 2 codes
length 15 bits: 0 codes
length 16 bits: 7 codes
total 40 codes
codeword 00 (0x0) (sz 2) -> 1 (0x1)
codeword 01 (0x1) (sz 2) -> 2 (0x2)
codeword 100 (0x4) (sz 3) -> 0 (0x0)
codeword 101 (0x5) (sz 3) -> 17 (0x11)
codeword 1100 (0xc) (sz 4) -> 3 (0x3)
[...]

SOS found (length 12 bytes)
Ns 3
component 0 (Y) Cs 1 Td 0 Ta 0
component 1 (Cb) Cs 2 Td 1 Ta 1
component 2 (Cr) Cs 3 Td 1 Ta 1
Ss 0 Se 63 Ah 0 Al 0
compressed pixeldata starts at pos 613

removing stuffing...
2489809 bytes with stuffing
2480908 data bytes without stuffing

parsing bitstream...
MCU 0 component 0 (Y) data_unit 0 Td 0 Ta 0
bitstream: decoded Huffman code 110 (0x6) (sz 3 bits): 7 (0x7), new bitpos is 3
decoded DC: -91
bitstream: decoded Huffman code 11001 (0x19) (sz 5 bits): 5 (0x5), new bitpos is 15
decoded AC: -25 (1 vals)
bitstream: decoded Huffman code 11010 (0x1a) (sz 5 bits): 18 (0x12), new bitpos is 25
decoded AC: 3 (3 vals)
bitstream: decoded Huffman code 100 (0x4) (sz 3 bits): 3 (0x3), new bitpos is 30
decoded AC: 4 (4 vals)
bitstream: decoded Huffman code 1010 (0xa) (sz 4 bits): 4 (0x4), new bitpos is 37
decoded AC: -13 (5 vals)
bitstream: decoded Huffman code 01 (0x1) (sz 2 bits): 2 (0x2), new bitpos is 43
decoded AC: -2 (6 vals)
bitstream: decoded Huffman code 100 (0x4) (sz 3 bits): 3 (0x3), new bitpos is 48
decoded AC: 4 (7 vals)
bitstream: decoded Huffman code 01 (0x1) (sz 2 bits): 2 (0x2), new bitpos is 53
decoded AC: -3 (8 vals)
bitstream: decoded Huffman code 100 (0x4) (sz 3 bits): 3 (0x3), new bitpos is 58
decoded AC: 4 (9 vals)
bitstream: decoded Huffman code 01 (0x1) (sz 2 bits): 2 (0x2), new bitpos is 63
decoded AC: -2 (10 vals)
bitstream: decoded Huffman code 100 (0x4) (sz 3 bits): 3 (0x3), new bitpos is 68
decoded AC: 5 (11 vals)
bitstream: decoded Huffman code 100 (0x4) (sz 3 bits): 3 (0x3), new bitpos is 74
decoded AC: 4 (12 vals)
bitstream: decoded Huffman code 01 (0x1) (sz 2 bits): 2 (0x2), new bitpos is 79
decoded AC: -2 (13 vals)
bitstream: decoded Huffman code 01 (0x1) (sz 2 bits): 2 (0x2), new bitpos is 83
decoded AC: 3 (14 vals)
bitstream: decoded Huffman code 00 (0x0) (sz 2 bits): 1 (0x1), new bitpos is 87
decoded AC: 1 (15 vals)
bitstream: decoded Huffman code 01 (0x1) (sz 2 bits): 2 (0x2), new bitpos is 90
decoded AC: -2 (16 vals)
bitstream: decoded Huffman code 01 (0x1) (sz 2 bits): 2 (0x2), new bitpos is 94
decoded AC: -2 (17 vals)
bitstream: decoded Huffman code 00 (0x0) (sz 2 bits): 1 (0x1), new bitpos is 98
decoded AC: -1 (18 vals)
bitstream: decoded Huffman code 1011 (0xb) (sz 4 bits): 17 (0x11), new bitpos is 103
decoded AC: 1 (20 vals)
bitstream: decoded Huffman code 1011 (0xb) (sz 4 bits): 17 (0x11), new bitpos is 108
decoded AC: 1 (22 vals)
bitstream: decoded Huffman code 11011 (0x1b) (sz 5 bits): 33 (0x21), new bitpos is 114
decoded AC: -1 (25 vals)
bitstream: decoded Huffman code 111010 (0x3a) (sz 6 bits): 65 (0x41), new bitpos is 121
decoded AC: -1 (30 vals)
bitstream: decoded Huffman code 00 (0x0) (sz 2 bits): 1 (0x1), new bitpos is 124
decoded AC: -1 (31 vals)
bitstream: decoded Huffman code 11111000 (0xf8) (sz 8 bits): 113 (0x71), new bitpos is 133
decoded AC: -1 (39 vals)
bitstream: decoded Huffman code 00 (0x0) (sz 2 bits): 1 (0x1), new bitpos is 136
decoded AC: 1 (40 vals)
bitstream: decoded Huffman code 00 (0x0) (sz 2 bits): 1 (0x1), new bitpos is 139
decoded AC: -1 (41 vals)
bitstream: decoded Huffman code 1011 (0xb) (sz 4 bits): 17 (0x11), new bitpos is 144
decoded AC: 1 (43 vals)
bitstream: decoded Huffman code 11000 (0x18) (sz 5 bits): 0 (0x0), new bitpos is 150
EOB, 43 AC values decoded
matrix (ZZ):
-91.000000 -25.000000 0.000000 3.000000 4.000000 -13.000000 -2.000000 4.000000 
-3.000000 4.000000 -2.000000 5.000000 4.000000 -2.000000 3.000000 1.000000 
-2.000000 -2.000000 -1.000000 0.000000 1.000000 0.000000 1.000000 0.000000 
0.000000 -1.000000 0.000000 0.000000 0.000000 0.000000 -1.000000 -1.000000 
0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 -1.000000 
1.000000 -1.000000 0.000000 1.000000 0.000000 0.000000 0.000000 0.000000 
0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 
0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 

matrix (ZZ reversed and dequantized):
-182.000000 -25.000000 -13.000000 -4.000000 9.000000 5.000000 0.000000 0.000000 
0.000000 4.000000 8.000000 -4.000000 -6.000000 0.000000 0.000000 0.000000 
6.000000 -6.000000 8.000000 -6.000000 -5.000000 -7.000000 -8.000000 7.000000 
8.000000 10.000000 -3.000000 0.000000 -6.000000 10.000000 0.000000 0.000000 
-4.000000 0.000000 0.000000 0.000000 -8.000000 0.000000 0.000000 0.000000 
3.000000 4.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 
0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 
0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 

matrix (after IDCT):
101.058870 99.134034 99.501082 99.745199 100.757630 104.739489 101.907223 91.720130 
108.421421 100.687807 96.749164 96.006366 94.644996 96.607602 99.323406 98.295197 
111.172128 102.882505 99.180743 101.597828 104.231434 105.820646 103.776522 98.659026 
105.287305 105.700023 107.016342 108.705911 112.355895 116.217755 112.120877 102.705886 
103.660071 107.318065 108.595667 105.548108 104.729464 109.549152 113.467439 112.992679 
104.362739 103.987812 105.457257 108.574719 110.084551 108.582550 106.347514 105.353124 
115.300599 109.434226 103.197912 102.226638 105.261276 107.450968 108.513365 109.807959 
108.340334 113.734391 112.786035 104.997920 100.853310 104.053282 110.159146 114.541289 

MCU 0 component 0 (Y) data_unit 1 Td 0 Ta 0
[...]
```

## Can you explain how JPEG works please?
Sorry, no. JPEG is quite complicated stuff, but you can find a lot of informations on the internet and the ITU-standards are freely available.  
Some links:
- https://en.wikipedia.org/wiki/JPEG_Interchange_Format
- https://en.wikipedia.org/wiki/JPEG_File_Interchange_Format
- https://github.com/corkami/formats/blob/master/image/JPEGRGB_dissected.png
- https://www.w3.org/Graphics/JPEG/jfif3.pdf
- https://www.w3.org/Graphics/JPEG/itu-t81.pdf
- https://en.wikipedia.org/wiki/YCbCr#JPEG_conversion
