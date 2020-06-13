# Reverse-engineered docs for the Loongson 3A4000 Crypto ASE

## Introduction

The [Loongson 3A4000][3a4000] (link in Chinese) has built-in crypto
acceleration support, but unfortunately, no public documentation has been
released despite repeated community requests.

This repository serves to consolidate the community effort of 3A4000 crypto
acceleration reverse-engineering.

As the instructions are present in the MIPS SDK too, it is assumed that the
crypto instructions actually belong to the upstream MSA specification,
although undocumented so far.

[3a4000]: http://www.loongson.cn/product/cpu/3/1288.html

## Source Materials

* The MIPS-provided [Codescape GNU Toolchain 2019.09-02][codescape]
* Loongnix's [binutils 2.28-13.fc21.loongson.5][loongnix-binutils] binary
* A Loongson 3A4000-powered system to experiment with

[codescape]: https://codescape.mips.com/components/toolchain/2019.09-02/downloads.html
[loongnix-binutils]: http://ftp.loongnix.org/os/loongnix/1.0/os/Packages/d/devtoolset-7-binutils-2.28-13.fc21.loongson.5.mips64el.rpm

The Loongson-modified binutils package has no source available at time of this writing.
The Codescape SDK has full sources available, though.

## Instructions

|Mnemonic|Instruction Description|
|--------|-----------------------|
|[`aes128.enc`](#aes128enc)|AES-128 Encrypt One Block|
|[`aes128.dec`](#aes128dec)|AES-128 Decrypt One Block|
|[`aes192.enc`](#aes192enc)|AES-192 Encrypt One Block|
|[`aes192.dec`](#aes192dec)|AES-192 Decrypt One Block|
|[`aes256.enc`](#aes256enc)|AES-256 Encrypt One Block|
|[`aes256.dec`](#aes256dec)|AES-256 Decrypt One Block|
|[`md5.4r`](#md54r)|MD5 Hash, 4 Rounds|
|[`md5.ms`](#md5ms)|MD5 Message Schedule|
|[`sha1.hash.4r`](#sha1hash4r)|SHA-1 Hash, 4 Rounds|
|[`sha1.ms.1`](#sha1ms1)|SHA-1 Message Schedule, First Half|
|[`sha1.ms.2`](#sha1ms2)|SHA-1 Message Schedule, Second Half|
|[`sha256.hash.2r`](#sha256hash2r)|SHA-256 Hash, 2 Rounds|
|[`sha256.ms.1`](#sha256ms1)|SHA-256 Message Schedule, First Half|
|[`sha256.ms.2`](#sha256ms2)|SHA-256 Message Schedule, Second Half|
|[`sha512.hash.r.1`](#sha512hashr1)|SHA-512 Hash, 1 Round, First Half|
|[`sha512.hash.r.2`](#sha512hashr2)|SHA-512 Hash, 1 Round, Second Half|
|[`sha512.ms.1`](#sha512ms1)|SHA-512 Message Schedule, First Half|
|[`sha512.ms.2`](#sha512ms2)|SHA-512 Message Schedule, Second Half|

There are some other crypto-related instructions present in the Codescape SDK binutils.
Behavior of these instructions are not fully known yet.

## Instruction Descriptions

### `aes128.enc`

```
OPCODE             WS    WD    MINOR_OP
011110 00000 00000 xxxxx xxxxx 010111
```

Format: `aes128.enc wd, ws`

Purpose: AES-128 Encrypt One Block

Operation:

```
plaintext <- WR[wd]
key <- WR[ws]
WR[wd] <- AES-128-ECB-Encrypt(plaintext, key)
```

### `aes128.dec`

```
OPCODE             WS    WD    MINOR_OP
011110 00000 00001 xxxxx xxxxx 010111
```

Format: `aes128.dec wd, ws`

Purpose: AES-128 Decrypt One Block

Operation:

```
ciphertext <- WR[wd]
key <- WR[ws]
WR[wd] <- AES-128-ECB-Decrypt(ciphertext, key)
```

### `aes192.enc`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 00010 xxxxx xxxxx xxxxx 010111
```

Format: `aes192.enc wd, ws, wt`

Purpose: AES-192 Encrypt One Block

Operation:

```
plaintext <- WR[wd]
key <- WR[ws] || WR[wt]_63..0
WR[wd] <- AES-192-ECB-Encrypt(plaintext, key)
```

### `aes192.dec`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 00011 xxxxx xxxxx xxxxx 010111
```

Format: `aes192.dec wd, ws, wt`

Purpose: AES-192 Decrypt One Block

Operation:

```
ciphertext <- WR[wd]
key <- WR[ws] || WR[wt]_63..0
WR[wd] <- AES-192-ECB-Decrypt(ciphertext, key)
```

### `aes256.enc`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 00100 xxxxx xxxxx xxxxx 010111
```

Format: `aes256.enc wd, ws, wt`

Purpose: AES-256 Encrypt One Block

Operation:

```
plaintext <- WR[wd]
key <- WR[ws] || WR[wt]
WR[wd] <- AES-256-ECB-Encrypt(plaintext, key)
```

### `aes256.dec`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 00101 xxxxx xxxxx xxxxx 010111
```

Format: `aes256.dec wd, ws, wt`

Purpose: AES-256 Decrypt One Block

Operation:

```
ciphertext <- WR[wd]
key <- WR[ws] || WR[wt]
WR[wd] <- AES-256-ECB-Decrypt(ciphertext, key)
```

### `md5.4r`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 10111 xxxxx xxxxx xxxxx 010111
```

Format: `md5.4r wd, ws, wt`

Purpose: MD5 Hash, 4 Rounds

Operation:

```
a <- WR[wt]_31..0
b <- WR[wt]_63..32
c <- WR[wt]_95..64
d <- WR[wt]_127..96

w0 <- WR[ws]_31..0
w1 <- WR[ws]_63..32
w2 <- WR[ws]_95..64
w3 <- WR[ws]_127..96

sel <- WR[wd]_5..0
if sel_1..0 != 0b00 then
    // behavior when sel is not multiple of 4 is not known yet
    UNKNOWN
endif

round <- sel >> 4

if round == 0 then
    a, b, c, d <- MD5_Round(FF, a, b, c, d, w0, 7)
    a, b, c, d <- MD5_Round(FF, a, b, c, d, w1, 12)
    a, b, c, d <- MD5_Round(FF, a, b, c, d, w2, 17)
    a, b, c, d <- MD5_Round(FF, a, b, c, d, w3, 22)
elseif round == 1 then
    a, b, c, d <- MD5_Round(GG, a, b, c, d, w0, 5)
    a, b, c, d <- MD5_Round(GG, a, b, c, d, w1, 9)
    a, b, c, d <- MD5_Round(GG, a, b, c, d, w2, 14)
    a, b, c, d <- MD5_Round(GG, a, b, c, d, w3, 20)
elseif round == 2 then
    a, b, c, d <- MD5_Round(HH, a, b, c, d, w0, 4)
    a, b, c, d <- MD5_Round(HH, a, b, c, d, w1, 11)
    a, b, c, d <- MD5_Round(HH, a, b, c, d, w2, 16)
    a, b, c, d <- MD5_Round(HH, a, b, c, d, w3, 23)
elseif round == 3 then
    a, b, c, d <- MD5_Round(II, a, b, c, d, w0, 6)
    a, b, c, d <- MD5_Round(II, a, b, c, d, w1, 10)
    a, b, c, d <- MD5_Round(II, a, b, c, d, w2, 15)
    a, b, c, d <- MD5_Round(II, a, b, c, d, w3, 21)
endif

WR[wd]_31..0   <- a
WR[wd]_63..32  <- b
WR[wd]_95..64  <- c
WR[wd]_127..96 <- d
```

### `md5.ms`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 11111 xxxxx xxxxx xxxxx 010111
```

Format: `md5.ms wd, ws, wt`

Purpose: MD5 Message Schedule

Operation:

```
sel <- WR[wd]_5..0

if sel == 0 or sel == 2 then
    WR[wd]_31..0   <- WR[ws]_31..0   + 0xd76aa478
    WR[wd]_63..32  <- WR[ws]_63..32  + 0xe8c7b756
    WR[wd]_95..64  <- WR[ws]_95..64  + 0x242070db
    WR[wd]_127..96 <- WR[ws]_127..96 + 0xc1bdceee
elseif sel == 1 or sel == 3 then
    WR[wd]_31..0   <- WR[ws]_31..0
    WR[wd]_63..32  <- WR[ws]_63..32
    WR[wd]_95..64  <- WR[ws]_95..64
    WR[wd]_127..96 <- WR[ws]_127..96
elseif sel == 4 or sel == 6 then
    WR[wd]_31..0   <- WR[wt]_31..0   + 0xf57c0faf
    WR[wd]_63..32  <- WR[wt]_63..32  + 0x4787c62a
    WR[wd]_95..64  <- WR[wt]_95..64  + 0xa8304613
    WR[wd]_127..96 <- WR[wt]_127..96 + 0xfd469501
elseif sel == 5 or sel == 7 then
    WR[wd]_31..0   <- WR[wt]_31..0
    WR[wd]_63..32  <- WR[wt]_63..32
    WR[wd]_95..64  <- WR[wt]_95..64
    WR[wd]_127..96 <- WR[wt]_127..96
elseif sel == 8 or sel == 10 then
    WR[wd]_31..0   <- WR[ws]_31..0   + 0x698098d8
    WR[wd]_63..32  <- WR[ws]_63..32  + 0x8b44f7af
    WR[wd]_95..64  <- WR[ws]_95..64  + 0xffff5bb1
    WR[wd]_127..96 <- WR[ws]_127..96 + 0x895cd7be
elseif sel == 9 or sel == 11 then
    WR[wd]_31..0   <- WR[ws]_31..0
    WR[wd]_63..32  <- WR[ws]_63..32
    WR[wd]_95..64  <- WR[ws]_95..64
    WR[wd]_127..96 <- WR[ws]_127..96
elseif sel == 12 or sel == 14 then
    WR[wd]_31..0   <- WR[wt]_31..0   + 0x6b901122
    WR[wd]_63..32  <- WR[wt]_63..32  + 0xfd987193
    WR[wd]_95..64  <- WR[wt]_95..64  + 0xa679438e
    WR[wd]_127..96 <- WR[wt]_127..96 + 0x49b40821
elseif sel == 13 or sel == 15 then
    WR[wd]_31..0   <- WR[wt]_31..0
    WR[wd]_63..32  <- WR[wt]_63..32
    WR[wd]_95..64  <- WR[wt]_95..64
    WR[wd]_127..96 <- WR[wt]_127..96
elseif sel == 18 then
    WR[wd]_31..0   <- WR[wd]_95..64  + 0xf61e2562
    WR[wd]_63..32  <- WR[wd]_63..32  + 0xc040b340
    WR[wd]_95..64  <- WR[ws]_127..96 + 0x265e5a51
    WR[wd]_127..96 <- WR[wd]_127..96 + 0xe9b6c7aa
elseif sel == 19 then
    WR[wd]_31..0   <- WR[wd]_95..64
    WR[wd]_95..64  <- WR[ws]_127..96
elseif sel == 22 then
    WR[wd]_31..0   <- WR[wd]_63..32  + 0xd62f105d
    WR[wd]_63..32  <- WR[ws]_95..64  + 0x02441453
    WR[wd]_95..64  <- WR[wt]_127..96 + 0xd8a1e681
    WR[wd]_127..96 <- WR[wd]_127..96 + 0xe7d3fbc8
elseif sel == 23 then
    WR[wd]_31..0   <- WR[wd]_63..32
    WR[wd]_63..32  <- WR[ws]_95..64
    WR[wd]_95..64  <- WR[wt]_127..96
elseif sel == 26 then
    WR[wd]_31..0   <- WR[ws]_63..32 + 0x21e1cde6
    WR[wd]_63..32  <- WR[wt]_95..64 + 0xc33707d6
    WR[wd]_95..64  <- WR[wd]_95..64 + 0xf4d50d87
    WR[wd]_127..96 <- WR[ws]_31..0  + 0x455a14ed
elseif sel == 27 then
    WR[wd]_31..0   <- WR[ws]_63..32
    WR[wd]_63..32  <- WR[wt]_95..64
    WR[wd]_127..96 <- WR[ws]_31..0
elseif sel == 30 then
    WR[wd]_31..0   <- WR[wt]_63..32 + 0xa9e3e905
    WR[wd]_63..32  <- WR[wd]_63..32 + 0xfcefa3f8
    WR[wd]_95..64  <- WR[wd]_95..64 + 0x676f02d9
    WR[wd]_127..96 <- WR[wt]_31..0  + 0x8d2a4c8a
elseif sel == 31 then
    WR[wd]_31..0   <- WR[wt]_63..32
    WR[wd]_127..96 <- WR[wt]_31..0
elseif sel == 34 then
    WR[wd]_31..0   <- WR[wd]_63..32  + 0xfffa3942
    WR[wd]_63..32  <- WR[ws]_31..0   + 0x8771f681
    WR[wd]_95..64  <- WR[ws]_127..96 + 0x6d9d6122
    WR[wd]_127..96 <- WR[wt]_95..64  + 0xfde5380c
elseif sel == 35 then
    WR[wd]_31..0   <- WR[wd]_63..32
    WR[wd]_63..32  <- WR[ws]_31..0
    WR[wd]_95..64  <- WR[ws]_127..96
    WR[wd]_127..96 <- WR[wt]_95..64
elseif sel == 38 then
    WR[wd]_31..0   <- WR[wd]_127..96 + 0xa4beea44
    WR[wd]_63..32  <- WR[wd]_63..32  + 0x4bdecfa9
    WR[wd]_95..64  <- WR[wd]_95..64  + 0xf6bb4b60
    WR[wd]_127..96 <- WR[ws]_95..64  + 0xbebfbc70
elseif sel == 39 then
    WR[wd]_31..0   <- WR[wd]_127..96
    WR[wd]_127..96 <- WR[ws]_95..64
elseif sel == 42 then
    WR[wd]_31..0   <- WR[wt]_63..32  + 0x289b7ec6
    WR[wd]_63..32  <- WR[wd]_63..32  + 0xeaa127fa
    WR[wd]_95..64  <- WR[wd]_95..64  + 0xd4ef3085
    WR[wd]_127..96 <- WR[wd]_127..96 + 0x04881d05
elseif sel == 43 then
    WR[wd]_31..0   <- WR[wt]_63..32
elseif sel == 46 then
    WR[wd]_31..0   <- WR[ws]_63..32  + 0xd9d4d039
    WR[wd]_63..32  <- WR[wt]_31..0   + 0xe6db99e5
    WR[wd]_95..64  <- WR[wt]_127..96 + 0x1fa27cf8
    WR[wd]_127..96 <- WR[wd]_127..96 + 0xc4ac5665
elseif sel == 47 then
    WR[wd]_31..0   <- WR[ws]_63..32
    WR[wd]_63..32  <- WR[wt]_31..0
    WR[wd]_95..64  <- WR[wt]_127..96
elseif sel == 50 then
    WR[wd]_31..0   <- WR[wd]_95..64  + 0xf4292244
    WR[wd]_63..32  <- WR[wd]_63..32  + 0x432aff97
    WR[wd]_95..64  <- WR[wt]_95..64  + 0xab9423a7
    WR[wd]_127..96 <- WR[wd]_127..96 + 0xfc93a039
elseif sel == 51 then
    WR[wd]_31..0   <- WR[wd]_95..64
    WR[wd]_95..64  <- WR[wt]_95..64
elseif sel == 54 then
    WR[wd]_31..0   <- WR[wt]_31..0   + 0x655b59c3
    WR[wd]_63..32  <- WR[wd]_63..32  + 0x8f0ccc92
    WR[wd]_95..64  <- WR[ws]_95..64  + 0xffeff47d
    WR[wd]_127..96 <- WR[wd]_127..96 + 0x85845dd1
elseif sel == 55 then
    WR[wd]_31..0   <- WR[wt]_31..0
    WR[wd]_95..64  <- WR[ws]_95..64
elseif sel == 58 then
    WR[wd]_31..0   <- WR[ws]_31..0   + 0x6fa87e4f
    WR[wd]_63..32  <- WR[wt]_127..96 + 0xfe2ce6e0
    WR[wd]_95..64  <- WR[wd]_95..64  + 0xa3014314
    WR[wd]_127..96 <- WR[wt]_63..32  + 0x4e0811a1
elseif sel == 59 then
    WR[wd]_31..0   <- WR[ws]_31..0
    WR[wd]_63..32  <- WR[wt]_127..96
    WR[wd]_127..96 <- WR[wt]_63..32
elseif sel == 62 then
    WR[wd]_31..0   <- WR[wd]_63..32  + 0xf7537e82
    WR[wd]_63..32  <- WR[ws]_127..96 + 0xbd3af235
    WR[wd]_95..64  <- WR[wd]_95..64  + 0x2ad7d2bb
    WR[wd]_127..96 <- WR[ws]_63..32  + 0xeb86d391
elseif sel == 63 then
    WR[wd]_31..0   <- WR[wd]_63..32
    WR[wd]_63..32  <- WR[ws]_127..96
    WR[wd]_127..96 <- WR[ws]_63..32
elseif sel == 16 or sel == 17 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[wt]_95..64
    WR[wd]_95..64  <- WR[ws]_63..32
    WR[wd]_127..96 <- WR[ws]_31..0
elseif sel == 20 or sel == 21 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[wt]_63..32
    WR[wd]_95..64  <- WR[ws]_63..32
    WR[wd]_127..96 <- WR[wt]_31..0
elseif sel == 24 or sel == 25 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[ws]_95..64
    WR[wd]_95..64  <- WR[ws]_127..96
    WR[wd]_127..96 <- WR[ws]_31..0
elseif sel == 28 or sel == 29 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[ws]_95..64
    WR[wd]_95..64  <- WR[wt]_127..96
    WR[wd]_127..96 <- WR[ws]_31..0
elseif sel == 32 or sel == 33 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[wt]_63..32
    WR[wd]_95..64  <- WR[wt]_127..96
    WR[wd]_127..96 <- WR[wt]_63..32
elseif sel == 36 or sel == 37 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[wt]_31..0
    WR[wd]_95..64  <- WR[wt]_127..96
    WR[wd]_127..96 <- WR[ws]_63..32
elseif sel == 40 or sel == 41 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[ws]_31..0
    WR[wd]_95..64  <- WR[ws]_127..96
    WR[wd]_127..96 <- WR[wt]_95..64
elseif sel == 44 or sel == 45 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[ws]_31..0
    WR[wd]_95..64  <- WR[ws]_127..96
    WR[wd]_127..96 <- WR[ws]_95..64
elseif sel == 48 or sel == 49 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[wt]_127..96
    WR[wd]_95..64  <- WR[ws]_31..0
    WR[wd]_127..96 <- WR[wt]_63..32
elseif sel == 52 or sel == 53 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[ws]_127..96
    WR[wd]_95..64  <- WR[ws]_31..0
    WR[wd]_127..96 <- WR[ws]_63..32
elseif sel == 56 or sel == 57 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[wt]_31..0
    WR[wd]_95..64  <- WR[wt]_95..64
    WR[wd]_127..96 <- WR[ws]_31..0
elseif sel == 60 or sel == 61 then
    WR[wd]_31..0   <- sel + 2
    WR[wd]_63..32  <- WR[wt]_31..0
    WR[wd]_95..64  <- WR[ws]_95..64
    WR[wd]_127..96 <- WR[ws]_31..0
endif
```

### `sha1.hash.4r`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 10000 xxxxx xxxxx xxxxx 010111
```

Format: `sha1.hash.4r wd, ws, wt`

Purpose: SHA-1 Hash, 4 Rounds

Description:

Performs 4 rounds of SHA-1. Round function and constant is built-in,
selectable by input operand.

Operation:

```
w0 <- WR[wt]_31..0
w1 <- WR[wt]_63..32
w2 <- WR[wt]_95..64
w3 <- WR[wt]_127..96

a <- WR[ws]_31..0
b <- WR[ws]_63..32
c <- WR[ws]_95..64
d <- WR[ws]_127..96
e <- WR[wd]_31..0

sel <- WR[wd]_65..64
dont_rotate_e <- WR[wd]_66

if not dont_rotate_e then
    e <- e rol 30
endif

if sel == 0 then
    f <- lambda b, c, d: (b and c) or ((not b) and d)
    k <- 0x5A827999
elseif sel == 1 then
    f <- lambda b, c, d: b xor c xor d
    k <- 0x6ED9EBA1
elseif sel == 2 then
    f <- lambda b, c, d: (b and c) or (b and d) or (c and d)
    k <- 0x8F1BBCDC
elseif sel == 3 then
    f <- lambda b, c, d: b xor c xor d
    k <- 0xCA62C1D6
endif

temp <- (a rol 5) + f(b, c, d) + e + k + w0
e <- d
d <- c
c <- b rol 30
b <- a
a <- temp

temp <- (a rol 5) + f(b, c, d) + e + k + w1
e <- d
d <- c
c <- b rol 30
b <- a
a <- temp

temp <- (a rol 5) + f(b, c, d) + e + k + w2
e <- d
d <- c
c <- b rol 30
b <- a
a <- temp

temp <- (a rol 5) + f(b, c, d) + e + k + w3
e <- d
d <- c
c <- b rol 30
b <- a
a <- temp

WR[wd]_0..31   <- a
WR[wd]_63..32  <- b
WR[wd]_95..64  <- c
WR[wd]_127..96 <- d
```

### `sha1.ms.1`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 11000 xxxxx xxxxx xxxxx 010111
```

Format: `sha1.ms.1 wd, ws, wt`

Purpose: SHA-1 Message Schedule, First Half

Operation:

```
WR[wd]_31..0   <- WR[wd]_31..0   xor WR[wd]_95..64  xor WR[wt]_31..0
WR[wd]_63..32  <- WR[wd]_63..32  xor WR[wd]_127..96 xor WR[wt]_63..32
WR[wd]_95..64  <- WR[wd]_95..64  xor WR[ws]_31..0   xor WR[wt]_95..64
WR[wd]_127..96 <- WR[wd]_127..96 xor WR[ws]_63..32  xor WR[wt]_127..96
```

### `sha1.ms.2`

```
OPCODE             WS    WD    MINOR_OP
011110 00000 11001 xxxxx xxxxx 010111
```

Format: `sha1.ms.2 wd, ws`

Purpose: SHA-1 Message Schedule, Second Half

Operation:

```
WR[wd]_31..0   <- (WR[wd]_31..0 rol 1)   xor (WR[ws]_63..32 rol 1)
WR[wd]_63..32  <- (WR[wd]_63..32 rol 1)  xor (WR[ws]_95..64 rol 1)
WR[wd]_95..64  <- (WR[wd]_95..64 rol 1)  xor (WR[ws]_127..96 rol 1)
WR[wd]_127..96 <- (WR[wd]_127..96 rol 1) xor (WR[wd]_31..0 rol 1)
```

Notes:

* The opcode of this instruction on Loongson 3A4000 is swapped with `aes.sr.dec` in upstream MIPS code.

### `sha256.hash.2r`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 10010 xxxxx xxxxx xxxxx 010111
```

Format: `sha256.hash.2r wd, ws, wt`

Purpose: SHA-256 Hash, 2 Rounds

Description:

Performs 2 rounds of SHA-256. K constants are not built in, these have to be
manually added prior to invocation.

Operation:

```
a <- WR[ws]_31..0
b <- WR[ws]_63..32
e <- WR[ws]_95..64
f <- WR[ws]_127..96

c <- WR[wd]_31..0
d <- WR[wd]_63..32
g <- WR[wd]_95..64
h <- WR[wd]_127..96

w0 <- WR[wt]_31..0
w1 <- WR[wt]_63..32

S0 <- lambda a: (a ror 2) xor (a ror 13) xor (a ror 22)
S1 <- lambda e: (e ror 6) xor (e ror 11) xor (e ror 25)
ch <- lambda e, f, g: (e and f) xor ((not e) and g)
maj <- lambda a, b, c: (a and b) xor (a and c) xor (b and c)

temp1 <- h + S1(e) + ch(e, f, g) + w0
temp2 <- S0(a) + maj(a, b, c)

h <- g
g <- f
f <- e
e <- d + temp1
d <- c
c <- b
b <- a
a <- temp1 + temp2

temp1 <- h + S1(e) + ch(e, f, g) + w1
temp2 <- S0(a) + maj(a, b, c)

h <- g
g <- f
f <- e
e <- d + temp1
d <- c
c <- b
b <- a
a <- temp1 + temp2

WR[wd]_0..31   <- a
WR[wd]_63..32  <- b
WR[wd]_95..64  <- e
WR[wd]_127..96 <- f
```

### `sha256.ms.1`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 11010 xxxxx xxxxx xxxxx 010111
```

Format: `sha256.ms.1 wd, ws, wt`

Purpose: SHA-256 Message Schedule, First Half

Operation:

```
a <- WR[ws]_31..0
b <- WR[ws]_63..32
e <- WR[ws]_95..64
f <- WR[ws]_127..96

e <- WR[wt]_31..0

s0 <- lambda x: (x ror 7) xor (x ror 18) xor (x >> 3)

WR[wd]_31..0   <- a + s0(b)
WR[wd]_63..32  <- b + s0(c)
WR[wd]_95..64  <- c + s0(d)
WR[wd]_127..96 <- d + s0(e)
```

### `sha256.ms.2`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 11011 xxxxx xxxxx xxxxx 010111
```

Format: `sha256.ms.2 wd, ws, wt`

Purpose: SHA-256 Message Schedule, Second Half

Operation:

```
x1 <- WR[ws]_63..32
x2 <- WR[ws]_95..64
x3 <- WR[ws]_127..96

y0 <- WR[wt]_31..0
y2 <- WR[wt]_95..64
y3 <- WR[wt]_127..96

z0 <- WR[wd]_31..0
z1 <- WR[wd]_63..32
z2 <- WR[wd]_95..64
z3 <- WR[wd]_127..96

s1 <- lambda x: (x ror 17) xor (x ror 19) xor (x >> 10)

z0 <- z0 + x1 + s1(y2)
z1 <- z1 + x2 + s1(y3)
z2 <- z2 + x3 + s1(z0)
z3 <- z3 + y0 + s1(z1)

WR[wd]_31..0   <- z0
WR[wd]_63..32  <- z1
WR[wd]_95..64  <- z2
WR[wd]_127..96 <- z3
```

### `sha512.hash.1`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 10100 xxxxx xxxxx xxxxx 010111
```

Format: `sha512.hash.1 wd, ws, wt`

Purpose: SHA-512 Hash, 1 Round, First Half

Operation:

```
S0 <- lambda a: (a ror 28) xor (a ror 34) xor (a ror 39)
S1 <- lambda e: (e ror 14) xor (e ror 18) xor (e ror 41)
ch <- lambda e, f, g: (e and f) xor ((not e) and g)
maj <- lambda a, b, c: (a and b) xor (a and c) xor (b and c)

WR[wd]_63..0   <- S0(WR[wd]_63..0)   + maj(WR[wd]_63..0,   WR[ws]_63..0,   WR[wt]_63..0)
WR[wd]_127..64 <- S1(WR[wd]_127..64) +  ch(WR[wd]_127..64, WR[ws]_127..64, WR[wt]_127..64)
```

### `sha512.hash.2`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 10101 xxxxx xxxxx xxxxx 010111
```

Format: `sha512.hash.2 wd, ws, wt`

Purpose: SHA-512 Hash, 1 Round, Second Half

Operation:

```
WR[wd]_63..0   <- WR[wd]_63..0   + WR[wd]_127..64 + WR[ws]_127..64 + WR[wt]_63..0
WR[wd]_127..64 <- WR[wd]_127..64 + WR[ws]_63..0   + WR[ws]_127..64 + WR[wt]_63..0
```

### `sha512.ms.1`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 11100 xxxxx xxxxx xxxxx 010111
```

Format: `sha512.ms.1 wd, ws, wt`

Purpose: SHA-512 Message Schedule, First Half

Operation:

```
s0 <- lambda x: (x ror 1) xor (x ror 8) xor (x >> 7)
s1 <- lambda x: (x ror 19) xor (x ror 61) xor (x >> 6)

WR[wd]_63..0   <- WR[wd]_63..0   + s1(WR[wt]_63..0)   + s0(WR[wd]_127..64)
WR[wd]_127..64 <- WR[wd]_127..64 + s1(WR[wt]_127..64) + s0(WR[ws]_63..0)
```

### `sha512.ms.2`

```
OPCODE       WT    WS    WD    MINOR_OP
011110 11101 xxxxx xxxxx xxxxx 010111
```

Format: `sha512.ms.2 wd, ws, wt`

Purpose: SHA-512 Message Schedule, Second Half

Operation:

```
WR[wd]_63..0   <- WR[wd]_63..0   + WR[ws]_127..64
WR[wd]_127..64 <- WR[wd]_127..64 + WR[wt]_63..0
```

## License

This document is placed under [CC0](./LICENSE).
