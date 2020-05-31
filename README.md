# Reverse-engineering the Loongson 3A4000 Crypto ASE

## Introduction

The [Loongson 3A4000][3a4000] (link in Chinese) has built-in crypto
acceleration support, but unfortunately, without public documentation despite
much community effort. This repository serves to consolidate the community
effort of 3A4000 crypto acceleration support reverse-engineering.

[3a4000]: http://www.loongson.cn/product/cpu/3/1288.html

## Source Materials

* The MIPS-provided [Codescape GNU Toolchain 2019.09-02][codescape]
* Loongnix's [binutils 2.28-13.fc21.loongson.5][loongnix-binutils] binary
* A Loongson 3A4000-powered system to experiment with

[codescape]: https://codescape.mips.com/components/toolchain/2019.09-02/downloads.html
[loongnix-binutils]: http://ftp.loongnix.org/os/loongnix/1.0/os/Packages/d/devtoolset-7-binutils-2.28-13.fc21.loongson.5.mips64el.rpm

The Loongson-modified binutils package has no source available.
It's certainly a shame, but there's little we can do to improve. The Codescape
SDK has full sources available, though.

## Instructions

To be filled.

## Instruction Descriptions

To be filled.

## License

This document is placed under [CC0](./LICENSE).
