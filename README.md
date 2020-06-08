# Reverse-engineered docs for the Loongson 3A4000 Crypto ASE

## Introduction

The [Loongson 3A4000][3a4000] (link in Chinese) has built-in crypto
acceleration support, but unfortunately, no public documentation has been
released despite repeated community requests.

This repository serves to consolidate the community effort of 3A4000 crypto
acceleration reverse-engineering.

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
|[`sha1.hash.4r`](#sha1hash4r)|SHA-1 Hash, 4 Rounds|
|[`sha1.ms.1`](#sha1ms1)|SHA-1 Message Schedule, First Half|
|[`sha1.ms.2`](#sha1ms2)|SHA-1 Message Schedule, Second Half|
|[`sha256.hash.2r`](#sha256hash2r)|SHA-256 Hash, 2 Rounds|
|[`sha256.ms.1`](#sha256ms1)|SHA-256 Message Schedule, First Half|
|[`sha256.ms.2`](#sha256ms2)|SHA-256 Message Schedule, Second Half|

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

## License

This document is placed under [CC0](./LICENSE).
