https://github.com/DrKLO/Telegram (v13.2.2 on master)
___
**Tools:** apktool, JADX, cve-bin-tool (+Syft, OSV-SCALIBR, LibRARIAN, Semantic Copycat BinarySniffer & MobSF)
**DBs:** NVD, CVE, OSV, Github Security Advisories
___
- APK downloaded from https://telegram.org/android
- APK extracted via `java -jar apktool d Telegram.apk` (got some warnings?? mb a non-issue)
- `file *.so` (under WSL2 Ubuntu) executed in `Telegram/lib/arm64-v8a`
```output
lnm@lnm-pc:/mnt/s/Files/Polytech/5/reverse/repos/R54/local/telegram/Telegram/lib/arm64-v8a$ file *.so
libdatastore_shared_counter.so: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, for Android 21, built by NDK r25c (9519653), BuildID[sha1]=17db37bd6770ac00dd2d1d2828839fd23a7959a3, stripped
liblanguage_id_l2c_jni.so:      ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, for Android 21, built by NDK r27 (12077973), BuildID[md5/uuid]=4e1cbc7b2a12b63992ad6bcaaa139a2b, stripped
libtmessages.49.so:             ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, for Android 23, built by NDK r27c (12479018), BuildID[sha1]=5359f7ec7e5327b2ccffea5d6c47c327464ea702, stripped
libtwidget.so:                  ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, for Android 23, built by NDK r27c (12479018), BuildID[sha1]=9157ba519bdfde83d97a6f005db6765c5f33370a, stripped
```
- `readelf -h libtmessages.49.so`
```output
lnm@lnm-pc:/mnt/s/Files/Polytech/5/reverse/repos/R54/local/telegram/Telegram/lib/arm64-v8a$ readelf -h libtmessages.49.so
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          64 (bytes into file)
  Start of section headers:          22134296 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         9
  Size of section headers:           64 (bytes)
  Number of section headers:         27
  Section header string table index: 26
```
- `readelf -d libtmessages.49.so`
```output
lnm@lnm-pc:/mnt/s/Files/Polytech/5/reverse/repos/R54/local/telegram/Telegram/lib/arm64-v8a$ readelf -d libtmessages.49.s
o

Dynamic section at offset 0x15087c0 contains 34 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libjnigraphics.so]
 0x0000000000000001 (NEEDED)             Shared library: [liblog.so]
 0x0000000000000001 (NEEDED)             Shared library: [libz.so]
 0x0000000000000001 (NEEDED)             Shared library: [libGLESv2.so]
 0x0000000000000001 (NEEDED)             Shared library: [libEGL.so]
 0x0000000000000001 (NEEDED)             Shared library: [libandroid.so]
 0x0000000000000001 (NEEDED)             Shared library: [libOpenSLES.so]
 0x0000000000000001 (NEEDED)             Shared library: [libm.so]
 0x0000000000000001 (NEEDED)             Shared library: [libdl.so]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so]
 0x000000000000000e (SONAME)             Library soname: [libtmessages.49.so]
 0x000000000000001e (FLAGS)              SYMBOLIC BIND_NOW
 0x000000006ffffffb (FLAGS_1)            Flags: NOW
 0x0000000000000007 (RELA)               0xd048
 0x0000000000000008 (RELASZ)             1216680 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000006ffffff9 (RELACOUNT)          50644
 0x0000000000000017 (JMPREL)             0x1360f0
 0x0000000000000002 (PLTRELSZ)           9120 (bytes)
 0x0000000000000003 (PLTGOT)             0x1517c78
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000006 (SYMTAB)             0x2f8
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000005 (STRTAB)             0x6130
 0x000000000000000a (STRSZ)              28435 (bytes)
 0x000000006ffffef5 (GNU_HASH)           0x5538
 0x0000000000000019 (INIT_ARRAY)         0x150c768
 0x000000000000001b (INIT_ARRAYSZ)       88 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x150c758
 0x000000000000001c (FINI_ARRAYSZ)       16 (bytes)
 0x000000006ffffff0 (VERSYM)             0x4e88
 0x000000006ffffffe (VERNEED)            0x54d4
 0x000000006fffffff (VERNEEDNUM)         3
 0x0000000000000000 (NULL)               0x0
```
